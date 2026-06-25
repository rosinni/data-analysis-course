# Clase 05 — Introducción a Pandas

> El DataFrame: la tabla que cambia todo

### ¿Qué vamos a ver hoy?

- Qué es Pandas y por qué existe
- Series: la columna como estructura
- DataFrame: la tabla como estructura
- Leer un archivo CSV con `pd.read_csv()`
- Selección básica de filas y columnas

---


Mira este código. Es exactamente el tipo de cosa que escribiste en la Clase 02 para agrupar ventas por categoría.

```python
ventas = [
    {"producto": "Notebook",  "categoria": "tech",    "precio": 320000},
    {"producto": "Mouse",     "categoria": "tech",    "precio": 8500},
    {"producto": "Escritorio","categoria": "muebles", "precio": 85000},
    {"producto": "Silla",     "categoria": "muebles", "precio": 45000},
]

totales = {}
for v in ventas:
    cat = v["categoria"]
    if cat not in totales:
        totales[cat] = 0
    totales[cat] += v["precio"]

print(totales)
# {'tech': 328500, 'muebles': 130000}
```

Funciona. Pero son seis líneas para calcular un total agrupado por categoría, lo cual es una de las operaciones más comunes en análisis de datos. Si después quisieras el promedio en lugar del total, tendrías que reescribir parte del bucle. Si quisieras ordenar el resultado, otra línea más. Cada pregunta nueva sobre los datos significa más código.

Pandas existe para que esas preguntas se respondan en una línea, no en seis.

### Qué es Pandas

Pandas es una librería de Python diseñada específicamente para trabajar con datos tabulares, es decir, filas y columnas, como una planilla de cálculo. No reemplaza lo que aprendiste hasta ahora; lo que aprendiste es la base que explica por qué Pandas funciona como funciona. Una columna de Pandas, en el fondo, se comporta de forma parecida a una lista. Una tabla de Pandas, en el fondo, se parece a una lista de diccionarios. La diferencia es que Pandas tiene funciones ya escritas para las operaciones que tú venías programando a mano.

### De dónde sale Pandas: instalar la librería

Antes de poder usar Pandas, hay algo que tienes que entender: Pandas no viene incluido cuando instalas Python. Python por sí solo trae herramientas básicas —las que usaste hasta ahora: `print()`, `len()`, listas, diccionarios— pero Pandas es una librería externa, escrita por otra gente, que tienes que instalar por separado antes de poder importarla.

Esto es distinto a lo que hiciste hasta ahora. Hasta esta clase, todo lo que usaste ya estaba disponible apenas abrías Python. A partir de ahora empezarás a usar herramientas que alguien construyó fuera del lenguaje base, y que eliges agregar a tu proyecto según lo que necesites. Para instalar Pandas, usas `pip`, que es el gestor de paquetes de Python — el programa que se encarga de buscar, descargar e instalar librerías. Se ejecuta desde la terminal (no desde un archivo `.py`, sino desde la línea de comandos de tu sistema):

```bash
pip install pandas
```

Una vez que ese comando termina, Pandas queda instalado en tu computadora y disponible para cualquier script de Python que quieras escribir, no solo para uno en particular. Lo instalas una vez, lo usas todas las veces que quieras.

> Si estás usando **Google Colab** o un entorno de notebooks en la nube provisto por tu institución, es muy probable que Pandas ya venga instalado de antemano y no necesites ejecutar nada de esto — directamente puedes importarlo. Si estás trabajando con **Anaconda**, también viene preinstalado junto con varias otras librerías de análisis de datos. El `pip install pandas` es necesario solamente si instalaste Python de la forma más básica, sin ninguna de estas distribuciones. Ante la duda, prueba directamente el `import pandas as pd` de la sección siguiente: si no da error, ya está instalado y puedes seguir sin instalar nada.

Para usar Pandas dentro de tu código, una vez instalado, hay que importarlo. Por convención, casi todo el mundo lo importa con el alias `pd`:

```python
import pandas as pd
```

`import` es la instrucción que le dice a Python "quiero usar esta librería en este archivo". `as pd` le pone un apodo más corto: en vez de escribir `pandas.algo()` cada vez, vas a escribir `pd.algo()`. Esa convención —`pd`— la vas a ver en absolutamente todo el código de Pandas que encuentres, en tutoriales, en documentación, en proyectos reales. Conviene adoptarla desde ya.

### Series: una columna con superpoderes

La estructura más simple de Pandas es la Series. Es muy parecida a una lista de Python, pero con algo extra: cada valor tiene un índice asociado, y la Series sabe hacer operaciones sobre todos sus valores a la vez, sin que tú escribas un bucle.

```python
precios = pd.Series([12000, 8500, 34000, 5200, 19900])
print(precios)
```

```
0    12000
1     8500
2    34000
3     5200
4    19900
dtype: int64
```

La columna de la izquierda es el índice — por defecto, empieza en 0, igual que las posiciones de una lista. La columna de la derecha son los valores. `dtype: int64` te dice el tipo de dato que contiene la Series — en este caso, números enteros.

Ahora mira esto, que es donde se nota la diferencia con una lista común:

```python
precios_con_iva = precios * 1.19
print(precios_con_iva)
```

```
0    14280.0
1    10115.0
2    40460.0
3     6188.0
4    23681.0
dtype: float64
```

Con una lista de Python, `precios * 1.19` no calcularía esto — multiplicaría la lista entera por 1.19 y daría un error, porque las listas no soportan esa operación. Con una Series, la multiplicación se aplica automáticamente a cada elemento. Esto se llama operación vectorizada: en lugar de escribir un bucle que recorra elemento por elemento, escribes la operación una sola vez y Pandas la aplica a toda la columna.

### DataFrame: la tabla completa

Una Series es una sola columna. Un DataFrame es la tabla completa — varias columnas combinadas, cada una con su nombre. Si trabajaste con una lista de diccionarios en las clases anteriores, ya tienes la intuición correcta: cada diccionario era una fila, y las claves eran como nombres de columnas. Un DataFrame formaliza exactamente esa idea.

```python
ventas = pd.DataFrame([
    {"producto": "Notebook",  "categoria": "tech",    "precio": 320000},
    {"producto": "Mouse",     "categoria": "tech",    "precio": 8500},
    {"producto": "Escritorio","categoria": "muebles", "precio": 85000},
    {"producto": "Silla",     "categoria": "muebles", "precio": 45000},
])

print(ventas)
```

```
    producto categoria  precio
0   Notebook      tech  320000
1      Mouse      tech    8500
2  Escritorio   muebles   85000
3       Silla   muebles   45000
```

Esa es la misma lista de diccionarios del ejemplo del principio, convertida en tabla. Cada clave del diccionario se volvió una columna. Cada diccionario se volvió una fila.

#### Inspeccionar un DataFrame

Cuando tienes una tabla nueva, las primeras preguntas suelen ser: ¿cuántas filas tiene?, ¿qué columnas hay?, ¿qué tipo de datos tiene cada una? Pandas tiene métodos para responder eso sin tener que imprimir toda la tabla:

```python
print(ventas.shape)       # (4, 4) -> 4 filas, 4 columnas
print(ventas.columns)     # Index(['producto', 'categoria', 'precio'], dtype='object')
print(ventas.dtypes)      # tipo de dato de cada columna
print(ventas.head(2))     # las primeras 2 filas
```

.`head(n)` es especialmente útil cuando trabajas con tablas grandes — te muestra solo las primeras filas, en lugar de imprimir miles de líneas en la pantalla.

### Leer datos desde un archivo CSV

Hasta ahora construiste los DataFrames a mano, escribiendo los datos directamente en el código. En la práctica, casi nunca harás eso. Los datos vendrán de un archivo — generalmente un CSV, que es básicamente una tabla guardada como texto, donde cada línea es una fila y los valores están separados por comas.

Imagina que tienes un archivo llamado `ventas.csv` con este contenido:

```
producto,categoria,precio,stock
Notebook,tech,320000,5
Mouse,tech,8500,42
Escritorio,muebles,85000,3
Silla,muebles,45000,8
Monitor,tech,210000,0
```

Para cargarlo como DataFrame, usas `pd.read_csv()`: 

```python
ventas = pd.read_csv("ventas.csv")
print(ventas)
```

```
     producto categoria  precio  stock
0    Notebook      tech  320000      5
1       Mouse      tech    8500     42
2  Escritorio   muebles   85000      3
3       Silla   muebles   45000      8
4     Monitor      tech  210000      0
```

Esa única línea —`pd.read_csv("ventas.csv")`— reemplaza todo lo que hubieras tenido que escribir a mano para leer el archivo, separar cada línea por comas, y armar la lista de diccionarios. Pandas hace todo eso por ti, y además detecta automáticamente qué tipo de dato tiene cada columna.

> Una aclaración importante: `pd.read_csv()` necesita la ruta correcta al archivo. Si el archivo está en la misma carpeta que tu script, alcanza con el nombre. Si está en otra carpeta, necesitas la ruta completa, igual que cuando abres un archivo con `open()`.

### Seleccionar columnas

Una vez que tienes el DataFrame, lo más básico que necesitarás es acceder a una columna específica. Se hace con corchetes, parecido a cómo accedías a una clave de un diccionario:

```python
print(ventas["precio"])
```

```
0    320000
1      8500
2     85000
3     45000
4    210000
Name: precio, dtype: int64
```

Eso te devuelve una Series — la columna `"precio"` sola. Si necesitas varias columnas a la vez, pasas una lista de nombres entre los corchetes:

```python
print(ventas[["producto", "precio"]])
```

```
     producto  precio
0    Notebook  320000
1       Mouse    8500
2  Escritorio   85000
3       Silla   45000
4     Monitor  210000
```

Observa el doble corchete: el corchete de afuera es la selección, el de adentro es la lista de nombres de columnas que quieres. Si te olvidas uno de los corchetes, Pandas no va a entender qué le estás pidiendo.

### Seleccionar filas

Para seleccionar filas por posición, usas `.iloc[]` (de "index location"):

```python
print(ventas.iloc[0])      # la primera fila
print(ventas.iloc[0:3])    # las primeras tres filas
```

`.iloc[0]` te devuelve la primera fila completa, con todas sus columnas. `.iloc[0:3]` funciona igual que el slicing que ya conocés de las listas — toma desde la posición 0 hasta la 2 inclusive (el 3 no se incluye, como en cualquier slice de Python).

Para seleccionar una fila por su valor de índice en lugar de su posición, existe `.loc[]`. Por ahora, mientras el índice sea el numérico por defecto, `.iloc` y `.loc` se comportan igual — la diferencia se nota más adelante, cuando el índice de un DataFrame no sea simplemente 0, 1, 2, 3...

### Todo junto: de la lista de diccionarios al DataFrame

Volvamos al ejemplo del principio del artículo. Esto es lo que hacías con un bucle:

```python
ventas = [
    {"producto": "Notebook",  "categoria": "tech",    "precio": 320000},
    {"producto": "Mouse",     "categoria": "tech",    "precio": 8500},
    {"producto": "Escritorio","categoria": "muebles", "precio": 85000},
    {"producto": "Silla",     "categoria": "muebles", "precio": 45000},
]

totales = {}
for v in ventas:
    cat = v["categoria"]
    if cat not in totales:
        totales[cat] = 0
    totales[cat] += v["precio"]
```

Con Pandas, el mismo resultado —aunque todavía no vimos cómo agrupar, lo verás en la próxima clase— empieza simplemente convirtiendo la lista en DataFrame y ya tienes acceso a todo lo que viste hoy:

```python
df = pd.DataFrame(ventas)

print(df.shape)
print(df["precio"])
print(df[["producto", "precio"]])
print(df.iloc[0])
```

Todavía no agrupaste nada — eso viene en la próxima clase con `.groupby()`. Pero ya puedes inspeccionar la tabla, seleccionar columnas específicas y acceder a filas, todo sin un solo bucle escrito por ti. Lo que antes eran varias líneas de lógica manual, ahora son llamadas directas a métodos que Pandas ya tiene resueltos.


## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `pip install pandas` | Instalar la librería antes de poder usarla |
| `import pandas as pd` | Importar la librería con su alias estándar |
| `pd.Series([...])` | Crear una columna individual con operaciones vectorizadas |
| `pd.DataFrame([...])` | Crear una tabla a partir de una lista de diccionarios |
| `pd.read_csv("archivo.csv")` | Cargar un archivo CSV como DataFrame |
| `.shape` | Ver la cantidad de filas y columnas |
| `.columns` | Ver los nombres de las columnas |
| `.dtypes` | Ver el tipo de dato de cada columna |
| `.head(n)` | Ver las primeras `n` filas |
| `df["columna"]` | Seleccionar una columna como Series |
| `df[["col1", "col2"]]` | Seleccionar varias columnas como DataFrame |
| `.iloc[]` | Seleccionar filas por posición |


## Recursos adicionales

- [Pandas Docs — Getting Started](https://pandas.pydata.org/docs/getting_started/index.html)
- [Pandas Docs — 10 minutes to pandas](https://pandas.pydata.org/docs/user_guide/10min.html)
- [Pandas Docs — read_csv](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html)
- [Real Python — Pandas DataFrames](https://realpython.com/pandas-dataframe/)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 04 — List Comprehensions y Limpieza de Strings](../clase-04/README.md) · [Módulo 2](../README.md) · Clase 06 — Filtrado y GroupBy →*