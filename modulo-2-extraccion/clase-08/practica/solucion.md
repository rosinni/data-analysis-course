# Solución — Clase 08: Exploración Inicial y Visualización Básica

> ⚠️ **Intentá resolver los ejercicios antes de mirar esto.**

### Ejercicio 1 — ¿Llegaron todos los datos?


Lo primero que hace un analista cuando recibe un DataFrame es verificar que la carga fue correcta. Antes de interpretar los datos necesitamos saber cuántos registros y cuántas columnas fueron recuperados.

### Código

```python
print(df.shape)
```

Salida esperada

```
(250, 5)
```

También podemos obtener cada valor por separado.

```python
print(df.shape[0])   # filas
print(df.shape[1])   # columnas
```

Salida

```
250
5
```

El DataFrame contiene 250 registros y 5 columnas. A simple vista, la extracción parece haberse realizado correctamente.


### Ejercicio 2 — Primera inspección visual

Antes de realizar cualquier cálculo es recomendable observar algunas filas para confirmar que los datos tienen sentido.

### Código

```python
df.head()
```

Salida aproximada

```
   id      nombre categoria     precio  stock
0   1  Mouse Gamer   gaming  260318.02     31
1   2      Headset   gaming   33167.77     54
2   3     Notebook     tech  197752.04     91
...
```

También es posible observar únicamente tres registros.

```python
df.head(3)
```

## ¿Qué deberíamos observar?

- Las columnas tienen nombres coherentes.
- Los IDs comienzan en 1.
- Los precios parecen expresados en pesos.
- El stock contiene números enteros.

Las primeras observaciones parecen representar correctamente productos del catálogo.


### Ejercicio 3 — Revisando el final del DataFrame

Algunos errores de exportación aparecen únicamente al final del archivo.

### Código

```python
df.tail()
```

## ¿Qué deberíamos observar?

- Los registros están completos.
- No existen filas truncadas.
- Los IDs continúan la secuencia.


No se observan problemas evidentes al final del DataFrame.


### Ejercicio 4 — Conociendo la estructura

### Código

```python
df.info()
```

## ¿Qué deberíamos observar?

La salida mostrará algo similar a:

```
RangeIndex: 250 entries

Data columns (total 5 columns)

id
nombre
categoria
precio
stock
```

También podremos identificar los tipos de datos.

- id → int64
- nombre → object
- categoria → object
- precio → float64
- stock → int64

La estructura del DataFrame coincide con lo esperado para un catálogo de productos.


### Ejercicio 5 — Auditoría de calidad

La función `info()` también permite descubrir si existen valores faltantes.

## ¿Qué deberíamos observar?

Al comparar el número de valores no nulos con la cantidad total de registros, veremos que:

- la columna `precio` posee algunos valores faltantes;
- la columna `categoria` también presenta registros sin información.


El dataset contiene valores nulos que deberán corregirse en la etapa de limpieza de datos.



### Ejercicio 6 — Entendiendo el catálogo

## Código

```python
df["categoria"].value_counts()
```

### ¿Qué deberíamos observar?

Obtendremos la cantidad de productos por categoría.

Esto permite identificar rápidamente cuáles concentran la mayor parte del catálogo.


## Ejercicio 7 — Participación porcentual

### Código

```python
df["categoria"].value_counts(normalize=True)
```

## ¿Qué deberíamos observar?

La salida mostrará la participación relativa de cada categoría.

Los valores estarán expresados como proporciones.

Por ejemplo:

```
tech          0.36
muebles       0.22
...
```


## Ejercicio 8 — Preparando una reunión ejecutiva

### Código

```python
df["categoria"].value_counts().plot(kind="bar")

plt.show()
```

### ¿Qué deberíamos observar?

Cada barra representa una categoría y su altura indica la cantidad de productos que pertenecen a ella.


## Ejercicio 9 — Un gráfico listo para presentar

### Código

```python
df["categoria"].value_counts().plot(kind="bar")

plt.title("Cantidad de productos por categoría")
plt.xlabel("Categoría")
plt.ylabel("Cantidad")

plt.show()
```

El gráfico ahora puede incorporarse a un informe porque incluye contexto suficiente para interpretarlo.

### Ejercicio 10 — Informe de exploración

## Posible informe

- El DataFrame contiene 250 registros y 5 columnas.
- Existen valores faltantes en las columnas `precio` y `categoria`.
- Predominan columnas de texto y numéricas.
- El catálogo está distribuido entre varias categorías, aunque algunas contienen más productos que otras.
- Se detectó al menos un precio extremadamente alto que podría corresponder a un error de carga.
- También existen algunos registros con stock negativo.
- Antes de realizar análisis más avanzados sería recomendable ejecutar un proceso de limpieza de datos.