# Clase 10 — Agrupaciones y Tablas Dinámicas

> Cuando el insight está escondido en los grupos

## ¿Qué vamos a ver hoy?

- Qué es un insight y por qué aparece al agrupar
- `groupby()` con funciones de agregación
- Múltiples columnas y múltiples métricas con `agg()`
- Tablas dinámicas con `pivot_table()`
- `unstack()` para reorganizar el resultado
- Visualización básica del resultado agrupado

---

Un insight es una conclusión no obvia que surge de analizar datos. No es simplemente "las ventas fueron $500.000 este mes", eso es un dato. Un insight es "las ventas en Córdoba cayeron un 30% en marzo, pero solo en la categoría electrónica, mientras que el resto del país subió", eso es algo que no veías mirando los números en bruto y que cambia una decisión. La mayoría de los insights no aparecen mirando filas individuales. Aparecen cuando agrupás los datos según algún criterio y calculás métricas sobre cada grupo. Una venta de $45.000 no te dice nada sola. El promedio de ventas de todos los vendedores de Rosario en el segundo trimestre sí te dice algo. Eso es exactamente lo que vas a aprender a hacer hoy.


> Para esta clase vas a trabajar con ventas de una cadena de retail con presencia en varias ciudades de Argentina. 

```python
df = pd.read_csv("ventas_retail.csv")
print(df.head())
print(df.dtypes)
```

500 filas, 7 columnas. Suficiente para que las preguntas que vas a responder hoy tengan sentido, pero manejable para entender qué está pasando.

### La pregunta que parece simple

El director de ventas te manda un mensaje: "¿cuánto vendió cada ciudad este año?". Tienes el dataset cargado. La pregunta es clara. Pero si miras el DataFrame, las ventas no están agrupadas por ciudad, hay 500 filas donde Buenos Aires, Córdoba y Rosario se mezclan en cualquier orden. Lo primero que se te ocurre es un bucle:

```python
df = pd.read_csv("ventas_retail.csv")

totales = {}
for i in range(len(df)):
    ciudad = df["ciudad"].iloc[i]
    monto  = df["monto"].iloc[i]
    if ciudad not in totales:
        totales[ciudad] = 0
    totales[ciudad] += monto

print(totales)
```

Funciona. Pero antes de que termines de escribirlo, llega otro mensaje: "y también el promedio, y el ticket máximo por ciudad". Ahora tienes que modificar el bucle para acumular tres métricas en lugar de una. Y cuando llegue la siguiente pregunta, "lo mismo pero por categoría", vuelves a escribir prácticamente el mismo código desde cero.

El problema no es que el bucle sea difícil. El problema es que para una operación que vas a necesitar docenas de veces en cualquier proyecto de análisis, estás escribiendo la misma lógica una y otra vez. `groupby()` existe para resolver exactamente eso.

### groupby(): la lógica detrás

`groupby()` no devuelve un DataFrame directamente (es importante que recuerdes esto), devuelve un objeto intermedio que representa los grupos, antes de que calcules nada sobre ellos. Ese objeto espera que le digas qué operación aplicar.

```python
grupos = df.groupby("ciudad")
print(type(grupos))
# <class 'pandas.core.groupby.DataFrameGroupBy'>
```

Para obtener un resultado, combinas `groupby()` con una función de agregación: `sum()`, `mean()`, `count()`, `max()`, `min()`.

```python
# Total de ventas por ciudad
df.groupby("ciudad")["monto"].sum()

# Promedio de ventas por ciudad
df.groupby("ciudad")["monto"].mean()

# Cantidad de ventas por ciudad
df.groupby("ciudad")["monto"].count()
```

```
ciudad
Buenos Aires    1823450
Córdoba         1654320
Mendoza         1432110
Rosario         1711890
Tucumán         1598230
Name: monto, dtype: int64
```

El resultado es una Series donde el índice son las ciudades y los valores son la métrica calculada. Puedes ordenarlo, filtrarlo, graficarlo, es un objeto de Pandas como cualquier otro.

#### Agrupar por múltiples columnas

No estás limitado a una sola columna de agrupación. Puedes agrupar por varias a la vez pasando una lista:

```python
df.groupby(["ciudad", "categoria"])["monto"].sum()
```

```
ciudad        categoria
Buenos Aires  Alimentos      342100
              Deportes       398200
              Electrónica    521300
              Hogar          287400
              Ropa           274450
Córdoba       Alimentos      ...
```

Ahora el índice tiene dos niveles, ciudad y categoría. Eso se llama MultiIndex, y es lo que vas a reorganizar con `unstack()` más adelante.

### Múltiples métricas con agg()

Hasta ahora calculaste una sola métrica por grupo. Con `.agg()` puedes calcular varias al mismo tiempo:

```python
df.groupby("vendedor")["monto"].agg(["sum", "mean", "count", "max"])
```

```
               sum          mean  count     max
vendedor
Ana Torres    892300  47489.36     18  148900
Carlos Vera   743200  38063.15     19  142100
...
```

También puedes darle nombres descriptivos a cada métrica y aplicar funciones distintas a columnas distintas:

```python
df.groupby("ciudad").agg(
    total_ventas   = ("monto",    "sum"),
    ticket_promedio= ("monto",    "mean"),
    unidades_total = ("unidades", "sum"),
    descuento_max  = ("descuento","max"),
)
```

```
              total_ventas  ticket_promedio  unidades_total  descuento_max
ciudad
Buenos Aires       1823450         72938.0             912           0.20
Córdoba            1654320         66172.8             834           0.20
...
```

Esa sintaxis `nombre_columna = ("columna_original", "función")`, es la forma más legible de usar `.agg()` cuando quieres controlar los nombres del resultado.

### reset_index(): volver a un DataFrame plano

El resultado de `groupby()` tiene el campo de agrupación como índice, no como columna. A veces eso es útil; otras veces quieres un DataFrame plano donde todo sea una columna normal.

```python
resultado = df.groupby("ciudad")["monto"].sum()
print(type(resultado.index))   # Index — el índice es la ciudad

resultado_plano = resultado.reset_index()
print(resultado_plano)
```

```
          ciudad    monto
0   Buenos Aires  1823450
1        Córdoba  1654320
2        Mendoza  1432110
3        Rosario  1711890
4        Tucumán  1598230
```

Ahora `ciudad` es una columna más, no el índice. Esto es lo que necesitas cuando quieres pasar el resultado a una función que espera un DataFrame normal, o guardarlo en un archivo.

### pivot_table(): la tabla dinámica de Pandas

`groupby()` agrupa por una o más columnas y devuelve una Serie o un objeto con el índice como agrupación. `pivot_table()` hace algo más visual: genera una tabla donde los valores de una columna se convierten en filas, los valores de otra se convierten en columnas, y en cada celda hay una métrica calculada, exactamente como una tabla dinámica de Excel.

```python
tabla = df.pivot_table(
    values  = "monto",
    index   = "ciudad",
    columns = "categoria",
    aggfunc = "sum",
)

print(tabla)
```

```
categoria     Alimentos  Deportes  Electrónica     Hogar      Ropa
ciudad
Buenos Aires     342100    398200       521300    287400    274450
Córdoba          298700    341200       487100    264800    262520
Mendoza          241300    298100       412300    234100    246310
Rosario          312400    367800       498200    274300    259190
Tucumán          287100    321400       456100    251900    281730
```

Cada fila es una ciudad, cada columna es una categoría, y cada celda es la suma de ventas de esa combinación. De un vistazo puedes ver cuál categoría domina en cada ciudad, algo que con `groupby()` te requería más trabajo para visualizar.

#### Valores nulos en pivot_table

Si alguna combinación de ciudad + categoría no tiene ningún registro en el dataset, la celda queda como `NaN`. Puedes rellenarla con un valor por defecto:

```python
tabla = df.pivot_table(
    values   = "monto",
    index    = "ciudad",
    columns  = "categoria",
    aggfunc  = "sum",
    fill_value = 0,
)
```

#### Múltiples métricas en pivot_table

```python
tabla_multiple = df.pivot_table(
    values  = "monto",
    index   = "ciudad",
    columns = "trimestre",
    aggfunc = ["sum", "mean"],
)

print(tabla_multiple)
```

Ahora el encabezado tiene dos niveles: primero la función (`sum` o `mean`) y después el trimestre. Es un MultiIndex en las columnas.

### unstack(): rotar un índice en columnas

Cuando tienes un resultado con MultiIndex, como el que genera `groupby()` con dos columnas, `unstack()` convierte el nivel más interno del índice en columnas, generando una tabla similar al resultado de `pivot_table()`.

```python
# groupby con dos columnas → MultiIndex
por_ciudad_trim = df.groupby(["ciudad", "trimestre"])["monto"].sum()
print(por_ciudad_trim)
```

```
ciudad        trimestre
Buenos Aires  Q1    412300
              Q2    489200
              Q3    521400
              Q4    400550
Córdoba       Q1    ...
```

```python
# unstack() convierte el trimestre (nivel interno) en columnas
tabla_unstacked = por_ciudad_trim.unstack()
print(tabla_unstacked)
```

```
trimestre         Q1      Q2      Q3      Q4
ciudad
Buenos Aires  412300  489200  521400  400550
Córdoba       387100  412300  445200  409720
Mendoza       321400  358200  392100  360410
Rosario       401200  428900  468700  413090
Tucumán       371200  398100  421300  407630
```

El resultado de `unstack()` es exactamente el mismo que obtendrías con `pivot_table()` en este caso son dos caminos distintos hacia la misma estructura. `pivot_table()` es más directo cuando sabés de antemano qué tabla quieres construir; `unstack()` es útil cuando ya tienes un resultado de `groupby()` y quieres reorganizarlo.

### Visualización básica del resultado

Un resultado agrupado es mucho más fácil de interpretar visualmente que como tabla de números. Pandas tiene integración directa con Matplotlib a través del método `.plot()`, que permite generar gráficos básicos sin importar Matplotlib explícitamente.

```python
import matplotlib.pyplot as plt

# Ventas totales por ciudad — gráfico de barras
totales_ciudad = df.groupby("ciudad")["monto"].sum().sort_values(ascending=False)

totales_ciudad.plot(kind="bar", figsize=(8, 4), title="Ventas totales por ciudad")
plt.ylabel("Monto ($)")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

Para la tabla dinámica, un gráfico de barras agrupadas muestra las categorías por ciudad:

```python
tabla.plot(kind="bar", figsize=(10, 5), title="Ventas por ciudad y categoría")
plt.ylabel("Monto ($)")
plt.xticks(rotation=45)
plt.legend(title="Categoría")
plt.tight_layout()
plt.show()
```

`.plot()` funciona directamente sobre el resultado de `groupby()` o sobre una `pivot_table()` porque ambos devuelven objetos de Pandas. El parámetro `kind` controla el tipo de gráfico: `"bar"` para barras verticales, `"barh"` para horizontales, `"line"` para líneas, `"pie"` para torta.

### Todo junto: responder preguntas reales

Volvamos a las preguntas del principio y respondámoslas con las herramientas de esta clase:

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("ventas_retail.csv")

# ¿Qué ciudad vende más?
print("=== Ciudad con más ventas ===")
por_ciudad = df.groupby("ciudad")["monto"].sum().sort_values(ascending=False)
print(por_ciudad)
print(f"\nGanadora: {por_ciudad.index[0]} (${por_ciudad.iloc[0]:,.0f})")

# ¿Qué categoría genera más ingresos por trimestre?
print("\n=== Ingresos por categoría y trimestre ===")
tabla_trim = df.pivot_table(
    values    = "monto",
    index     = "categoria",
    columns   = "trimestre",
    aggfunc   = "sum",
    fill_value= 0,
)
print(tabla_trim)

# ¿Qué vendedor tiene el ticket promedio más alto?
print("\n=== Ticket promedio por vendedor ===")
ticket = df.groupby("vendedor")["monto"].mean().sort_values(ascending=False)
print(ticket.round(0))
print(f"\nMejor ticket promedio: {ticket.index[0]} (${ticket.iloc[0]:,.0f})")

# Visualización
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

por_ciudad.plot(kind="barh", ax=axes[0], title="Ventas totales por ciudad")
axes[0].set_xlabel("Monto ($)")

ticket.plot(kind="bar", ax=axes[1], title="Ticket promedio por vendedor")
axes[1].set_ylabel("Monto promedio ($)")
axes[1].tick_params(axis="x", rotation=45)

plt.tight_layout()
plt.show()
```

Tres preguntas de negocio respondidas con tres llamadas a `groupby()` y una `pivot_table()`. Sin bucles, sin diccionarios acumuladores, sin quince líneas de lógica manual.

---

## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| Insight | Conclusión no obvia que surge de analizar datos agrupados |
| `groupby("col")` | Agrupar filas según los valores de una columna |
| `groupby(["col1","col2"])` | Agrupar por múltiples columnas — genera MultiIndex |
| `.sum()` / `.mean()` / `.count()` | Funciones de agregación sobre el grupo |
| `.agg({...})` | Calcular múltiples métricas en una sola operación |
| `.reset_index()` | Convertir el índice de agrupación en columna normal |
| `pivot_table()` | Tabla dinámica: una columna como filas, otra como columnas, métrica en las celdas |
| `fill_value=0` | Reemplazar NaN en pivot_table por un valor por defecto |
| `unstack()` | Convertir el nivel interno de un MultiIndex en columnas |
| `.plot(kind="bar")` | Gráfico de barras directo desde un resultado de Pandas |

---

## Recursos adicionales

- [Pandas Docs — GroupBy](https://pandas.pydata.org/docs/user_guide/groupby.html)
- [Pandas Docs — pivot_table](https://pandas.pydata.org/docs/reference/api/pandas.pivot_table.html)
- [Real Python — Pandas GroupBy](https://realpython.com/pandas-groupby/)
- [Pandas Docs — Visualization](https://pandas.pydata.org/docs/user_guide/visualization.html)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 09 — Limpieza de Datos](../clase-09/README.md) · [Módulo 3](../README.md) · Clase 11 — Estadística Descriptiva →*