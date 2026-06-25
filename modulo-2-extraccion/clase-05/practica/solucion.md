# Solución — Clase 05: Introducción a Pandas

> ⚠️ **Intenta resolver los ejercicios antes de mirar esto.**


## Ejercicio 1 — Una Series y su diferencia con una lista

```python
edades = [22, 35, 28, 41, 19]
edades_en_5_anios = edades + 5
# TypeError: can only concatenate list (not "int") to list
```

Con una lista, el operador `+` no significa "sumar a cada elemento" — significa "concatenar". Python espera que sumes otra lista, no un número, y por eso falla.

```python
import pandas as pd

edades = pd.Series([22, 35, 28, 41, 19])

edades_en_5_anios = edades + 5

print(edades_en_5_anios)
```

```
0    27
1    40
2    33
3    46
4    24
dtype: int64
```

**La respuesta a la pregunta:** una lista de Python no sabe qué hacer con `+ 5` porque las listas no tienen ninguna noción de "aplicar una operación a cada elemento" — el operador `+` está definido para concatenar, nada más. Una Series sí define ese comportamiento: cuando le aplicás una operación matemática, Pandas la reparte automáticamente entre todos sus valores. Eso es lo que se llama una operación vectorizada, y es una de las razones centrales por las que Pandas existe.


## Ejercicio 2 — Tu primer DataFrame

```python
import pandas as pd

consultas = [
    {"mascota": "Toby",   "especie": "perro", "edad": 4, "peso_kg": 18.5},
    {"mascota": "Mishi",  "especie": "gato",  "edad": 2, "peso_kg": 4.1},
    {"mascota": "Rocky",  "especie": "perro", "edad": 7, "peso_kg": 25.0},
    {"mascota": "Luna",   "especie": "gato",  "edad": 1, "peso_kg": 3.8},
]

df = pd.DataFrame(consultas)

print(df)
print(df.shape)
```

```
  mascota especie  edad  peso_kg
0    Toby   perro     4     18.5
1   Mishi    gato     2      4.1
2   Rocky   perro     7     25.0
3    Luna    gato     1      3.8

(4, 4)
```

**Respuesta:** el DataFrame tiene 4 filas y 4 columnas. `pd.DataFrame()` toma cada diccionario de la lista como una fila, y las claves —que en este caso son iguales en todos los diccionarios— se convierten en los nombres de columna.


## Ejercicio 3 — Inspeccionar antes de trabajar

```python
import pandas as pd

productos = pd.DataFrame([
    {"nombre": "Notebook",   "precio": 320000, "stock": 5},
    {"nombre": "Mouse",      "precio": 8500,   "stock": 42},
    {"nombre": "Teclado",    "precio": 45000,  "stock": 12},
    {"nombre": "Monitor",    "precio": 210000, "stock": 0},
    {"nombre": "Auriculares","precio": 32000,  "stock": 18},
])

print(productos.shape)
# (5, 3)

print(productos.columns)
# Index(['nombre', 'precio', 'stock'], dtype='object')

print(productos.dtypes)
# nombre    object
# precio     int64
# stock      int64
# dtype: object

print(productos.head(3))
#       nombre  precio  stock
# 0   Notebook  320000      5
# 1      Mouse    8500     42
# 2    Teclado   45000     12
```

**Nota sobre `dtypes`:** la columna `nombre` aparece como `object`. En Pandas, los textos (strings) se representan con ese tipo genérico, no con algo como `str`. Vas a ver `object` muy seguido cuando inspecciones columnas de texto.


## Ejercicio 4 — El error de un solo corchete

```python
import pandas as pd

productos = pd.DataFrame([
    {"nombre": "Notebook", "precio": 320000, "stock": 5},
    {"nombre": "Mouse",    "precio": 8500,   "stock": 42},
    {"nombre": "Teclado",  "precio": 45000,  "stock": 12},
])

resultado_a = productos["precio"]
resultado_b = productos[["precio"]]
resultado_c = productos[["nombre", "precio"]]

print(type(resultado_a))   # <class 'pandas.core.series.Series'>
print(type(resultado_b))   # <class 'pandas.core.frame.DataFrame'>
print(type(resultado_c))   # <class 'pandas.core.frame.DataFrame'>
```

**La respuesta a la pregunta:** `resultado_a` es una Series, `resultado_b` es un DataFrame. La diferencia está en qué le pasás entre los corchetes externos. `productos["precio"]` recibe directamente el nombre de una columna como string — y eso devuelve esa columna como Series. `productos[["precio"]]` recibe una *lista* que contiene un solo nombre de columna — y cuando le pasás una lista, sin importar si tiene un elemento o varios, Pandas siempre te devuelve un DataFrame, no una Series.

Es una distinción sutil pero importante: el tipo de resultado no depende de cuántas columnas pediste, depende de si las pediste como string suelto o como lista. `df["col"]` → Series. `df[["col"]]` → DataFrame de una sola columna.


## Ejercicio 5 — Seleccionar filas con iloc

```python
import pandas as pd

notas = pd.DataFrame([
    {"alumno": "Ana",    "nota": 8.5},
    {"alumno": "Luis",   "nota": 6.0},
    {"alumno": "Clara",  "nota": 9.2},
    {"alumno": "Pedro",  "nota": 5.5},
    {"alumno": "María",  "nota": 7.8},
    {"alumno": "Jorge",  "nota": 4.0},
])

primera_fila = notas.iloc[0]

rango_medio = notas.iloc[2:5]

ultima_fila = notas.iloc[-1]

print(primera_fila)
print(rango_medio)
print(ultima_fila)
```

```
alumno     Ana
nota       8.5
Name: 0, dtype: object

  alumno  nota
2  Clara   9.2
3  Pedro   5.5
4  María   7.8

alumno    Jorge
nota        4.0
Name: 5, dtype: object
```

**Sobre el rango medio:** para obtener las posiciones 2, 3 y 4 (Clara, Pedro, María), el slice es `2:5` — igual que con listas, el slicing toma desde el primer número hasta el segundo sin incluirlo. `2:5` incluye las posiciones 2, 3 y 4, pero no la 5.

**Sobre el índice negativo:** `.iloc[-1]` funciona exactamente igual que con las listas que usaste en clases anteriores. `-1` es siempre la última fila, sin que necesites saber cuántas filas tiene el DataFrame.


## Ejercicio 6 — De CSV a respuestas

```python
import pandas as pd

inventario = pd.read_csv("inventario.csv")

print(inventario.shape)
# (5, 4)

print(inventario["titulo"])
# 0        Cien años de soledad
# 1                     Rayuela
# 2                   Ficciones
# 3    La casa de los espíritus
# 4                    El Aleph
# Name: titulo, dtype: object

print(inventario[["titulo", "precio"]])
#                      titulo  precio
# 0     Cien años de soledad   18500
# 1                   Rayuela   15200
# 2                 Ficciones   12800
# 3 La casa de los espíritus   16900
# 4                  El Aleph   11500

print(inventario.iloc[2])
# titulo     Ficciones
# autor      Jorge Luis Borges
# precio     12800
# stock      0
# Name: 2, dtype: object
```

**La respuesta a la pregunta de reflexión:** si `inventario.csv` no estuviera en la misma carpeta que el script, `pd.read_csv("inventario.csv")` no lo encontraría y lanzaría un `FileNotFoundError`. Tendrías que indicar la ruta completa hacia el archivo, por ejemplo `pd.read_csv("datos/inventario.csv")` si estuviera dentro de una carpeta llamada `datos`, o la ruta absoluta del sistema si está en otro lugar completamente distinto. Es el mismo concepto de rutas que se aplica a cualquier archivo que abras desde código, no es algo específico de Pandas.


## Ejercicio 7 — De bucle a Pandas

```python
import pandas as pd

productos = [
    {"nombre": "Notebook",   "categoria": "tech",    "precio": 320000, "stock": 5},
    {"nombre": "Escritorio", "categoria": "muebles", "precio": 85000,  "stock": 3},
    {"nombre": "Mouse",      "categoria": "tech",    "precio": 8500,   "stock": 42},
    {"nombre": "Silla",      "categoria": "muebles", "precio": 45000,  "stock": 8},
    {"nombre": "Monitor",    "categoria": "tech",    "precio": 210000, "stock": 0},
]

# 1. Convertir a DataFrame
df = pd.DataFrame(productos)

# 2. Equivalente a las listas nombres y precios, sin bucle
nombres = df["nombre"]
precios = df["precio"]

print(nombres)
print(precios)

# 3. Equivalente a la variable "primero"
primero = df.iloc[0]
print(primero)
```

**Sobre la parte 2:** fijate que `nombres` y `precios` ahora son Series, no listas de Python. Para la mayoría de los usos dentro de Pandas, eso no cambia nada — podés seguir imprimiéndolas, recorrerlas, operarlas. Si en algún momento necesitaras una lista de Python pura, existe el método `.tolist()`: `df["nombre"].tolist()` te devolvería exactamente `['Notebook', 'Escritorio', 'Mouse', 'Silla', 'Monitor']`.

**Sobre la parte 4 — filtrar por categoría:**

Si lo intentaste y no te salió, es exactamente lo esperado. Con las herramientas de hoy —selección de columnas y `.iloc` por posición— no hay forma directa de decir "dame solo las filas donde la categoría sea tech", porque ninguna de las dos te permite filtrar *según el contenido* de una columna. `.iloc` selecciona por posición numérica, no por condición.

Lo que necesitás es algo que evalúe una condición sobre toda la columna a la vez —parecido a lo que hiciste en el Ejercicio 1 con `edades + 5`, pero con una comparación en lugar de una suma— y use ese resultado para filtrar filas. Esa herramienta existe y se llama indexación booleana. Es exactamente el tema central de la próxima clase, junto con `.groupby()` para agrupar por categoría sin escribir el diccionario acumulador a mano como hacías hasta ahora.

---

← [Volver a ejercicios](./notebook.ipynb)