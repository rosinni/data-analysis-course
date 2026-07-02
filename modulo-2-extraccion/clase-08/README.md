# Clase 08 — Exploración de Datos (EDA - Exploratory Data Analysis)

> Conocer un dataset antes de empezar a analizarlo

> 🤔 **Para pensar antes de leer:** En la clase anterior aprendiste a traer datos desde una base de datos directamente a un DataFrame utilizando `pd.read_sql()`. Pero imagina que esa consulta devuelve 800.000 registros y 75 columnas. ¿Cómo sabrías si todos los datos llegaron correctamente? ¿Cómo descubrirías qué tipo de información contiene cada columna, si hay datos faltantes o si los valores tienen sentido? Antes de comenzar cualquier análisis, un analista dedica unos minutos a conocer el dataset que acaba de recibir.


## ¿Qué vamos a ver hoy?

- Cómo inspeccionar rápidamente un DataFrame
- Conocer su tamaño y estructura
- Obtener estadísticas descriptivas
- Contar categorías y frecuencias
- Crear los primeros gráficos con Matplotlib


Hasta ahora aprendiste dos formas de obtener datos. En las primeras clases cargaste archivos CSV utilizando `pd.read_csv()`. Después aprendiste a consultar bases de datos con SQL y finalmente a traer esos resultados directamente a un DataFrame mediante `pd.read_sql()`. Pero una vez que los datos llegan a Python aparece un problema completamente distinto. Ya no se trata de **cómo obtener los datos**, sino de **entender qué acabas de recibir**. Piensa en un médico que recibe una radiografía. Antes de hacer un diagnóstico necesita observar la imagen, identificar qué partes está viendo y verificar que la radiografía corresponda al paciente correcto. Con los datos ocurre exactamente lo mismo.

Cuando un analista recibe un DataFrame nuevo, nunca empieza calculando promedios o creando gráficos complejos. Lo primero que hace es inspeccionarlo. Se hace preguntas como:

- ¿Cuántas filas tiene?
- ¿Cuántas columnas?
- ¿Cómo se llaman?
- ¿Qué tipo de datos guarda cada una?
- ¿Hay valores vacíos?
- ¿Las primeras filas tienen sentido?

Responder esas preguntas suele llevar apenas unos segundos, pero puede evitar horas de trabajo sobre datos incorrectos. Por eso existe una etapa conocida como **Exploración Inicial de Datos** (*Initial Data Exploration*), cuyo objetivo no es sacar conclusiones, sino familiarizarse con el dataset antes de comenzar cualquier análisis.


## Un DataFrame desconocido

Durante esta clase vamos a utilizar el mismo DataFrame obtenido desde SQLite en la clase anterior.

```python
import sqlite3
import pandas as pd

conexion = sqlite3.connect("ventas.db")

df = pd.read_sql(
    "SELECT * FROM productos",
    conexion
)
```

A partir de este punto ya no importa si los datos vinieron de un archivo CSV, de una base de datos SQLite o incluso de un servidor remoto. Para Pandas siempre es exactamente lo mismo: un DataFrame y todas las herramientas que vas a aprender hoy funcionan sobre cualquier DataFrame, sin importar de dónde provengan los datos.


## ¿Cuántos datos tengo?

La primera pregunta suele ser la más sencilla. **¿Qué tamaño tiene este dataset?** No alcanza con mirar la pantalla, un DataFrame puede tener millones de filas y solamente estar viendo las primeras cinco, para responder esa pregunta existe el atributo `shape`.

```python
print(df.shape)
```

Salida:

```
(7, 5)
```

Ese resultado es una tupla, el primer número representa la cantidad de filas, el segundo representa la cantidad de columnas. En este ejemplo el DataFrame tiene siete registros y cinco columnas. Podemos acceder a cada uno por separado.

```python
filas = df.shape[0]
columnas = df.shape[1]

print(filas)
print(columnas)
```

Salida:

```
7
5
```

Es muy común que un analista revise `shape` apenas recibe un dataset. Si esperaba encontrar un millón de registros y `shape` devuelve solamente cien, probablemente hubo un problema durante la carga de datos. Por eso esta suele ser una de las primeras líneas que aparecen en cualquier notebook de análisis.


## Ver las primeras filas

Ya conocés esta función desde la introducción a Pandas.

```python
df.head()
```

Salida:

```
   id      nombre categoria   precio  stock
0   1   Notebook      tech  320000.0      5
1   2      Mouse      tech    8500.0     42
2   3 Escritorio   muebles   85000.0      3
3   4      Silla   muebles   45000.0      8
4   5    Monitor      tech  210000.0      0
```

`head()` devuelve las primeras cinco filas del DataFrame, no modifica los datos, ni elimina información. Simplemente muestra una pequeña muestra para que puedas ver rápidamente cómo está organizada la tabla. Si necesitas otra cantidad de filas, podés indicarla como parámetro.

```python
df.head(3)
```

Salida:

```
   id      nombre categoria   precio  stock
0   1   Notebook      tech  320000.0      5
1   2      Mouse      tech    8500.0     42
2   3 Escritorio   muebles   85000.0      3
```

Cuando un DataFrame tiene cientos de miles de registros, `head()` permite inspeccionar los datos sin necesidad de recorrer toda la tabla.

## Ver las últimas filas

A veces el problema no está al principio del archivo, sino al final. Para eso existe `tail()`.

```python
df.tail()
```

Salida:

```
   id       nombre categoria   precio  stock
2   3  Escritorio   muebles   85000.0      3
3   4       Silla   muebles   45000.0      8
4   5     Monitor      tech  210000.0      0
5   6 Auriculares      tech   32000.0     18
6   7    Lámpara   muebles   12000.0     25
```

Funciona exactamente igual que `head()`, pero mostrando las últimas filas del DataFrame. También acepta un número.

```python
df.tail(2)
```

```
        nombre categoria precio stock
5  Auriculares      tech 32000    18
6      Lámpara  muebles 12000    25
```

Muchas veces los errores aparecen únicamente al final de un archivo. Por ejemplo, puede haberse cortado una exportación y las últimas filas estar incompletas. Ver tanto el principio como el final del DataFrame suele dar una idea bastante buena de cómo luce el conjunto de datos.


## Conocer la estructura del DataFrame

Hasta ahora vimos el contenido. Pero todavía no sabemos qué tipo de información guarda cada columna. ¿`precio` es un número?, ¿`stock` también?, ¿`nombre` es texto?. Para responder estas preguntas Pandas ofrece una de las funciones más útiles de toda la biblioteca.

```python
df.info()
```

Salida:

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 7 entries, 0 to 6
Data columns (total 5 columns):

 #   Column      Non-Null Count  Dtype
 0   id          7 non-null      int64
 1   nombre      7 non-null      object
 2   categoria   7 non-null      object
 3   precio      7 non-null      float64
 4   stock       7 non-null      int64

dtypes: float64(1), int64(2), object(2)
```

Aunque al principio parece mucha información, cada parte tiene un significado importante. La primera línea indica que estamos trabajando con un DataFrame. Después aparece la cantidad de registros, luego vemos una tabla donde cada fila representa una columna. Las columnas más importantes son:

- **Column**: nombre de la columna.
- **Non-Null Count**: cuántos valores no están vacíos.
- **Dtype**: tipo de dato que utiliza Pandas.

Si una columna tiene menos valores que la cantidad total de filas, significa que existen datos faltantes. Detectar esto al principio puede ahorrar muchos problemas más adelante.


## Obtener estadísticas descriptivas

Después de conocer la estructura, normalmente queremos una idea general de los valores numéricos. Para eso existe `describe()`.

```python
df.describe()
```

La salida será similar a esta:

```
              id         precio      stock
count   7.000000       7.000000   7.000000
mean    4.000000  101785.710000  14.428571
std     ...
min     ...
25%     ...
50%     ...
75%     ...
max     ...
```

`describe()` calcula automáticamente distintas estadísticas para todas las columnas numéricas. Entre ellas encontramos:

- **count**: cantidad de valores.
- **mean**: promedio.
- **std**: desviación estándar.
- **min**: valor mínimo.
- **25%**: primer cuartil.
- **50%**: mediana.
- **75%**: tercer cuartil.
- **max**: valor máximo.

No es necesario memorizar todas estas estadísticas por ahora. Lo importante es entender que `describe()` ofrece un resumen rápido del comportamiento de las columnas numéricas. Con una sola línea podés descubrir, por ejemplo, cuál es el precio más alto, el más bajo y cuál es el valor promedio de los productos almacenados. Muchas veces estas estadísticas alcanzan para detectar errores evidentes. Si la columna `edad` tiene un valor máximo de 850 años, probablemente exista un problema en los datos. Por eso `describe()` forma parte de las primeras funciones que utiliza prácticamente cualquier analista al recibir un nuevo dataset.

### Contar categorías con `value_counts()`

Hasta ahora exploramos principalmente columnas numéricas, pero muchos datasets también contienen columnas de texto que representan categorías, como ciudades, países, tipos de productos o estados de un pedido. En esos casos, calcular un promedio no tiene ningún sentido. ¿Cuál sería el promedio entre "tech" y "muebles"? Ninguno. Lo que suele interesar es otra cosa: **¿cuántas veces aparece cada categoría?**. Para responder esa pregunta Pandas ofrece `value_counts()`.

Supongamos que queremos saber cuántos productos pertenecen a cada categoría.

```python
df["categoria"].value_counts()
```

Salida:

```
categoria
tech        4
muebles     3
Name: count, dtype: int64
```

La salida indica que existen cuatro productos de tecnología y tres de muebles. Esta función cuenta automáticamente cuántas veces aparece cada valor distinto dentro de una columna. Es una herramienta extremadamente utilizada porque muchas preguntas reales consisten justamente en contar categorías.

Por ejemplo:

- ¿Cuántos clientes pertenecen a cada provincia?
- ¿Cuántos pedidos están pendientes?
- ¿Cuántos empleados trabajan en cada departamento?
- ¿Cuántos productos pertenecen a cada categoría?

Todas esas preguntas pueden responderse utilizando `value_counts()`. También es posible obtener los resultados como porcentajes.

```python
df["categoria"].value_counts(normalize=True)
```

Salida:

```
categoria
tech       0.571429
muebles    0.428571
```

Como esos valores representan proporciones, es habitual multiplicarlos por cien para interpretarlos como porcentajes. En este ejemplo, aproximadamente el 57 % de los productos pertenece a la categoría "tech", mientras que el 43 % corresponde a "muebles".


## Hasta ahora vimos números. El cerebro entiende mucho mejor las imágenes.

Imagina que alguien te muestra esta información.

```
tech        4
muebles     3
```

La entiendes sin problemas. Ahora imagina que aparecen veinte categorías distintas.

```
Notebook      53
Monitor       41
Mouse         98
Teclado       74
Auriculares   65
...
```

Todavía podés leer la información, pero cada vez cuesta más descubrir patrones. Los seres humanos somos muy buenos interpretando imágenes. Nuestro cerebro detecta mucho más rápido cuál es la barra más alta de un gráfico que cuál es el número más grande dentro de una lista. Por eso la visualización de datos es una parte tan importante del análisis.

Los gráficos no reemplazan a los números, los complementan. Permiten descubrir relaciones, diferencias y tendencias que muchas veces pasan desapercibidas cuando solamente observamos tablas.

## Matplotlib: la biblioteca para crear gráficos

Python posee varias bibliotecas para crear gráficos. La más utilizada históricamente es **Matplotlib**. Muchas otras bibliotecas, como Seaborn o Pandas, construyen sus propios gráficos utilizando Matplotlib por debajo, para utilizarla normalmente se importa su módulo `pyplot`.

```python
import matplotlib.pyplot as plt
```

Por convención casi todos los ejemplos que vas a encontrar en Internet utilizan el alias `plt`, no es obligatorio. Podría llamarse de cualquier otra forma, sin embargo, usar `plt` facilita leer ejemplos escritos por otras personas.


## Nuestro primer gráfico de barras

Volvamos al resultado de `value_counts()`.

```python
df["categoria"].value_counts()
```

Ese resultado puede transformarse directamente en un gráfico.

```python
df["categoria"].value_counts().plot(kind="bar")

plt.show()
```

Obtendremos un gráfico similar a este.

```
|
|████
|████
|████        ███
|████        ███
+---------------------
 tech     muebles
```

Cada barra representa una categoría. La altura de la barra indica cuántos elementos pertenecen a esa categoría. No tuvimos que calcular absolutamente nada, pandas utilizó el resultado de `value_counts()` y Matplotlib se encargó de dibujarlo. Este tipo de gráfico resulta ideal cuando queremos comparar cantidades entre distintas categorías. Por ejemplo:

- cantidad de alumnos por curso;
- ventas por sucursal;
- pedidos por estado;
- productos por categoría.

Siempre que el objetivo sea comparar grupos independientes, un gráfico de barras suele ser una muy buena elección.


## Agregar títulos y etiquetas

Aunque el gráfico anterior funciona, todavía le falta contexto. Si alguien lo observa sin conocer el dataset, difícilmente sepa qué representan las barras. Podemos agregar un título y nombres a los ejes.

```python
df["categoria"].value_counts().plot(kind="bar")

plt.title("Cantidad de productos por categoría")
plt.xlabel("Categoría")
plt.ylabel("Cantidad")

plt.show()
```

Ahora el gráfico es mucho más fácil de interpretar. Agregar títulos y etiquetas es una buena práctica, especialmente cuando el gráfico va a formar parte de un informe o una presentación. Un gráfico debería poder entenderse por sí solo, sin necesidad de que alguien lo explique.


## Visualizar distribuciones con histogramas

Los gráficos de barras sirven para comparar categorías, pero ¿qué ocurre cuando queremos estudiar una columna numérica?. Supongamos que queremos observar cómo se distribuyen los precios de nuestros productos. Para eso existe el histograma.

```python
df["precio"].plot(kind="hist")

plt.show()
```

Obtendremos un gráfico donde el eje horizontal representa rangos de precios y el eje vertical indica cuántos productos caen dentro de cada rango. A diferencia del gráfico de barras, aquí no estamos comparando categorías, estamos observando cómo se distribuyen los valores de una variable numérica.

Los histogramas permiten responder preguntas como:

- ¿La mayoría de los productos son baratos o caros?
- ¿Los salarios están concentrados alrededor de un valor?
- ¿Existen pocos valores extremadamente grandes?
- ¿La distribución parece uniforme o muy desigual?

Más adelante aprenderás que conocer la distribución de una variable es una de las tareas más importantes del análisis exploratorio. Por ahora alcanza con entender que el histograma es una herramienta diseñada para visualizar variables numéricas.


## Todo junto

Una exploración inicial suele combinar varias de las herramientas vistas hoy.

```python
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt

conexion = sqlite3.connect("ventas.db")

df = pd.read_sql(
    "SELECT * FROM productos",
    conexion
)

print(df.shape)

print(df.head())

print(df.info())

print(df.describe())

print(df["categoria"].value_counts())

df["categoria"].value_counts().plot(kind="bar")

plt.title("Productos por categoría")
plt.xlabel("Categoría")
plt.ylabel("Cantidad")

plt.show()

conexion.close()
```

En apenas unas pocas líneas logramos responder preguntas fundamentales sobre el dataset. Sabemos cuántas filas tiene, conocemos sus columnas, identificamos los tipos de datos, obtuvimos estadísticas descriptivas y contamos categorías. Finalmente construimos una visualización sencilla para resumir parte de la información. Este flujo representa el comienzo de prácticamente cualquier proyecto de análisis de datos.

Antes de limpiar datos, entrenar modelos de aprendizaje automático o construir tableros de control, un analista necesita conocer el dataset con el que va a trabajar.


## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `shape` | Obtener la cantidad de filas y columnas del DataFrame |
| `head()` | Ver las primeras filas |
| `tail()` | Ver las últimas filas |
| `info()` | Conocer la estructura y tipos de datos |
| `describe()` | Obtener estadísticas descriptivas de las columnas numéricas |
| `value_counts()` | Contar la frecuencia de cada categoría |
| `matplotlib.pyplot` | Biblioteca para crear gráficos |
| `plot(kind="bar")` | Crear un gráfico de barras |
| `plot(kind="hist")` | Crear un histograma |
| `plt.show()` | Mostrar el gráfico en pantalla |


## Recursos adicionales

- https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.info.html
- https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html
- https://matplotlib.org/stable/tutorials/index.html
- https://pandas.pydata.org/docs/user_guide/visualization.html


## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)


*← Clase 07 — Python + SQL · Módulo 2 · Clase 09 — Limpieza de Datos con Pandas →*

