# Clase 09 — Limpieza de Datos: Nulos, Duplicados y Tipos

> Los datos del mundo real nunca llegan limpios. Esta clase enseña a manejar eso.

> 🤔 **Para pensar antes de leer:** Imaginá que recibís un archivo de registros de empleados con 10.000 filas. Cuando lo abrís, encontrás que algunos tienen el campo de salario vacío, otros tienen el mismo empleado registrado dos veces, y los salarios de algunas filas llegaron como texto (`"45000"`) en lugar de número. ¿Podrías calcular el promedio salarial así? ¿Qué harías antes de empezar a analizar?


## ¿Qué vamos a ver hoy?

- Detectar y tratar valores nulos con `.isnull()`, `.dropna()`, `.fillna()`
- Detectar y eliminar duplicados con `.duplicated()`, `.drop_duplicates()`
- Convertir tipos de datos con `.astype()`
- Traer datos sucios desde SQLite y limpiarlos con Pandas


En las clases anteriores trabajaste con datos que llegaban prolijos, las columnas tenían los tipos correctos, no había celdas vacías, no había filas repetidas. Eso no es lo que pasa en la realidad.

Mirá esto. Es un fragmento de un dataset real de registros de personal:

```
nombre,ciudad,salario,fecha_ingreso,departamento
Ana Torres,Buenos Aires,45000,2021-03-15,Sistemas
Luis Gómez,,32000,2021-07-22,Contabilidad
Ana Torres,Buenos Aires,45000,2021-03-15,Sistemas
Clara Díaz,Rosario,38000,,Recursos Humanos
Pedro Ruiz,Córdoba,,2020-11-08,
María López,Mendoza,41000,2022-01-30,Sistemas
Luis Gómez,Córdoba,32000,2021-07-22,Contabilidad
```

Hay tres problemas visibles en esas siete filas: celdas vacías (`ciudad` de Luis, `fecha_ingreso` de Clara, `salario` y `departamento` de Pedro), filas duplicadas (Ana Torres aparece dos veces exactamente igual), y una fila casi duplicada (Luis Gómez aparece dos veces con distinta ciudad). Antes de poder analizar nada, tenés que decidir qué hacés con cada uno de esos problemas.

Eso es limpieza de datos. No es glamoroso, pero es la mayor parte del trabajo real de un analista.

### El dataset que acaba de llegar

Imaginá que el área de Recursos Humanos de una empresa te pasa acceso a su base de datos. Te dicen que tiene los registros de todos los empleados activos e inactivos, y que necesitan un análisis de salarios por departamento. Lo primero que hacés es conectarte y mirar qué hay adentro.

> Antes de continuar, ejecutá este script una sola vez para crear la base de datos con la que vamos a trabajar:

```python
import sqlite3

conexion = sqlite3.connect("rrhh.db")
cursor = conexion.cursor()

cursor.executescript("""
    CREATE TABLE IF NOT EXISTS empleados (
        id              INTEGER PRIMARY KEY,
        nombre          TEXT,
        ciudad          TEXT,
        departamento    TEXT,
        salario         TEXT,
        fecha_ingreso   TEXT,
        activo          TEXT
    );

    INSERT INTO empleados VALUES
        (1,  'Ana Torres',    'Buenos Aires', 'Sistemas',          '45000', '2021-03-15', '1'),
        (2,  'Luis Gómez',    NULL,           'Contabilidad',      '32000', '2021-07-22', '1'),
        (3,  'Ana Torres',    'Buenos Aires', 'Sistemas',          '45000', '2021-03-15', '1'),
        (4,  'Clara Díaz',    'Rosario',      'Recursos Humanos',  '38000', NULL,         '0'),
        (5,  'Pedro Ruiz',    'Córdoba',      NULL,                NULL,    '2020-11-08', '1'),
        (6,  'María López',   'Mendoza',      'Sistemas',          '41000', '2022-01-30', '1'),
        (7,  'Luis Gómez',    'Córdoba',      'Contabilidad',      '32000', '2021-07-22', '1'),
        (8,  'Sofía Herrera', 'Buenos Aires', 'Marketing',         '37500', '2023-05-12', NULL),
        (9,  'Carlos Vera',   NULL,           'Sistemas',          '52000', '2019-08-20', '1'),
        (10, 'Lucía Paz',     'Rosario',      'Recursos Humanos',  '29000', '2022-09-01', '0'),
        (11, 'Ana Torres',    'Buenos Aires', 'Sistemas',          '45000', '2021-03-15', '1'),
        (12, 'Martín Rojas',  'Mendoza',      'Contabilidad',      NULL,    '2021-04-18', '1'),
        (13, 'Julia Soto',    'Córdoba',      'Marketing',         '35000', NULL,         '1'),
        (14, 'Carlos Vera',   'Buenos Aires', 'Sistemas',          '52000', '2019-08-20', '1'),
        (15, 'Pedro Ruiz',    'Córdoba',      NULL,                NULL,    '2020-11-08', '1');
""")

conexion.commit()
conexion.close()
print("Base de datos rrhh.db creada.")
```

Ahora lo traés a Pandas tal como lo recibiste, sin tocar nada:

```python
import sqlite3
import pandas as pd

conexion = sqlite3.connect("rrhh.db")
df = pd.read_sql("SELECT * FROM empleados", conexion)
conexion.close()

print(df)
print(df.dtypes)
```

```
    id          nombre         ciudad    departamento salario fecha_ingreso activo
0    1      Ana Torres   Buenos Aires        Sistemas   45000    2021-03-15      1
1    2      Luis Gómez           None    Contabilidad   32000    2021-07-22      1
2    3      Ana Torres   Buenos Aires        Sistemas   45000    2021-03-15      1
3    4      Clara Díaz        Rosario Recursos Humanos  38000          None      0
4    5      Pedro Ruiz        Córdoba            None    None    2020-11-08      1
...

id               int64
nombre          object
ciudad          object
departamento    object
salario         object
fecha_ingreso   object
activo          object
dtype: object
```

Tres cosas llaman la atención de inmediato. Hay `None` en varias celdas — campos que deberían tener un valor y no lo tienen. Ana Torres aparece en la fila 0 y en la fila 2 con exactamente los mismos datos — un duplicado. Y `salario` figura como `object`, que en Pandas significa texto: el sistema de origen guardó los números como si fueran palabras, y mientras no lo corrijás, no podés hacer ninguna operación matemática con esa columna.

Esos son los tres problemas que vas a aprender a resolver en esta clase. Vamos por partes.

### Parte 1 — Valores nulos

#### Qué es un valor nulo

Un valor nulo es la ausencia de un dato. En Pandas se representa como `NaN` (para columnas numéricas) o `None` (para columnas de texto), pero ambos significan lo mismo: ese campo no tiene valor.

Los nulos aparecen por muchas razones: el campo era opcional en el formulario original, hubo un error de importación, el sistema de origen no tenía ese dato disponible. Lo importante es que **no podés operar con nulos sin decidir primero qué hacer con ellos** — sumar una columna con nulos da un resultado distinto al que esperás, y filtrar sin considerar los nulos puede hacerte perder filas que necesitabas.

#### Detectar nulos

El primer paso es saber exactamente cuántos nulos hay y dónde:

```python
print(df.isnull().sum())
```

```
id               0
nombre           0
ciudad           2
departamento     2
salario          2
fecha_ingreso    2
activo           1
dtype: int64
```

`.isnull()` devuelve un DataFrame de `True`/`False` — `True` donde hay nulo, `False` donde hay dato. `.sum()` suma los `True` de cada columna (recordá que `True` equivale a `1` en Python), dando la cantidad de nulos por columna.

Para ver qué porcentaje del total representa cada uno:

```python
porcentaje = (df.isnull().sum() / len(df) * 100).round(1)
print(porcentaje)
```

```
ciudad          13.3
departamento    13.3
salario         13.3
fecha_ingreso   13.3
activo           6.7
```

Saber el porcentaje importa porque la decisión de qué hacer con los nulos depende de cuántos hay. Un 1% de nulos en una columna y un 40% son situaciones muy distintas.

#### Eliminar filas con nulos: dropna()

La opción más agresiva es eliminar directamente las filas que tienen algún nulo:

```python
df_sin_nulos = df.dropna()
print(f"Filas originales: {len(df)}")
print(f"Filas sin nulos:  {len(df_sin_nulos)}")
```

```
Filas originales: 15
Filas sin nulos:  7
```

Perdiste casi la mitad de las filas. Eso puede ser aceptable o catastrófico dependiendo del análisis. Si solo necesitás los registros completos, `dropna()` está bien. Si perder esas filas sesga tu análisis, necesitás otra estrategia.

Podés ser más selectivo: eliminar solo las filas que tienen nulo en columnas específicas:

```python
# Solo eliminar si el salario es nulo — esas filas son inutilizables para análisis salarial
df_sin_salario_nulo = df.dropna(subset=["salario"])
print(len(df_sin_salario_nulo))
```

#### Rellenar nulos: fillna()

La otra opción es reemplazar los nulos por un valor que tenga sentido en el contexto:

```python
# Rellenar ciudad desconocida con un texto explícito
df["ciudad"] = df["ciudad"].fillna("Sin datos")

# Rellenar departamento con el más común (moda)
departamento_mas_comun = df["departamento"].value_counts().iloc[0]
df["departamento"] = df["departamento"].fillna(departamento_mas_comun)
```

No hay una regla universal sobre cuándo eliminar y cuándo rellenar. Depende de qué representa ese dato y qué análisis vas a hacer. Un salario nulo que reemplazás con `0` va a distorsionar cualquier cálculo de promedio. Una ciudad nula que reemplazás con `"Sin datos"` es inofensiva si solo querés contar empleados por ciudad (los que no tienen ciudad quedan agrupados en esa categoría).

### Parte 2 — Duplicados

#### Detectar duplicados

Un duplicado es una fila que repite exactamente los mismos valores que otra. En el dataset que cargaste, Ana Torres aparece tres veces con exactamente los mismos datos:

```python
print(df.duplicated().sum())
```

```
2
```

`.duplicated()` devuelve una Series de `True`/`False` — `True` para cada fila que ya apareció antes. La primera ocurrencia siempre es `False`; las repeticiones son `True`. Por eso con tres filas iguales el resultado es `2`, no `3`.

Para ver cuáles son esas filas:

```python
print(df[df.duplicated()])
```

También podés buscar duplicados considerando solo algunas columnas — útil cuando no importa si el `id` es distinto, sino si el nombre y la fecha son iguales:

```python
print(df.duplicated(subset=["nombre", "fecha_ingreso"]).sum())
```

#### Eliminar duplicados

```python
df_sin_duplicados = df.drop_duplicates()
print(f"Filas originales:       {len(df)}")
print(f"Sin duplicados exactos: {len(df_sin_duplicados)}")
```

Por defecto, `drop_duplicates()` conserva la primera ocurrencia y elimina las siguientes. Si querés conservar la última en cambio, pasás `keep='last'`.

Igual que con `dropna()`, podés aplicarlo solo sobre columnas específicas:

```python
# Conservar solo un registro por nombre + fecha_ingreso
df_sin_dup_nombre = df.drop_duplicates(subset=["nombre", "fecha_ingreso"])
```

Esto maneja el caso de Luis Gómez: aparece dos veces con la misma fecha pero distinta ciudad. Si definís que el duplicado se detecta por `nombre + fecha_ingreso`, se elimina una de las dos, aunque no sean idénticas.

### Parte 3 — Cambio de tipos: astype()

Ahora el tercer problema. `salario` llegó como texto desde SQLite porque así estaba declarada la columna. Antes de poder calcular cualquier cosa con salarios, hay que convertirla a número.

```python
print(df["salario"].dtype)    # object — es texto
print(df["salario"] + 1000)   # TypeError: no se puede sumar texto con número
```

`.astype()` convierte una columna al tipo que le indicás:

```python
df["salario"] = df["salario"].astype(float)
print(df["salario"].dtype)    # float64
```

Pero hay un problema: si la columna tiene nulos, `.astype(float)` falla porque no puede convertir `None` a número. Hay que manejar los nulos primero, o usar un tipo especial de entero que admite nulos:

```python
# Opción 1: eliminar nulos antes de convertir
df_limpio = df.dropna(subset=["salario"])
df_limpio["salario"] = df_limpio["salario"].astype(float)

# Opción 2: convertir con pd.to_numeric(), que maneja nulos y errores
df["salario"] = pd.to_numeric(df["salario"], errors="coerce")
```

`pd.to_numeric()` con `errors="coerce"` convierte los valores que puede y reemplaza los que no puede (nulos, textos no numéricos) con `NaN` — en lugar de lanzar un error que detiene el programa. Es la opción más segura cuando no tenés garantías sobre la calidad de los datos.

Lo mismo aplica para `activo`, que llegó como texto `"1"` y `"0"` y debería ser booleano:

```python
df["activo"] = df["activo"].map({"1": True, "0": False})
print(df["activo"].dtype)    # bool
```

`.map()` aplica un diccionario de reemplazo sobre cada valor de la columna — igual que el patrón que ya viste para convertir `"True"`/`"False"` en la Clase 01, pero ahora aplicado a una columna completa de Pandas.

### Todo junto: un pipeline de limpieza

Este es el flujo completo que vas a repetir en cualquier proyecto real. La idea es aplicar las transformaciones en un orden que tenga sentido: primero entender qué hay, después limpiar duplicados, después manejar nulos, después convertir tipos.

```python
import sqlite3
import pandas as pd

# 1. Traer los datos
conexion = sqlite3.connect("rrhh.db")
df = pd.read_sql("SELECT * FROM empleados", conexion)
conexion.close()

print("=== ESTADO INICIAL ===")
print(f"Filas:  {len(df)}")
print(f"Nulos:\n{df.isnull().sum()}")
print(f"Duplicados: {df.duplicated().sum()}")
print(f"Tipos:\n{df.dtypes}")

# 2. Eliminar duplicados exactos
df = df.drop_duplicates()
print(f"\nTras drop_duplicates: {len(df)} filas")

# 3. Manejar nulos
df["ciudad"] = df["ciudad"].fillna("Sin datos")
df["departamento"] = df["departamento"].fillna("Sin asignar")
df = df.dropna(subset=["salario"])   # sin salario no podemos analizar

# 4. Convertir tipos
df["salario"] = pd.to_numeric(df["salario"], errors="coerce")
df["activo"]  = df["activo"].map({"1": True, "0": False})

print("\n=== ESTADO FINAL ===")
print(f"Filas:  {len(df)}")
print(f"Nulos:\n{df.isnull().sum()}")
print(f"Tipos:\n{df.dtypes}")
print(f"\nSalario máximo: ${df['salario'].max():,.0f}")
print(f"Salario mínimo: ${df['salario'].min():,.0f}")
```

```
=== ESTADO INICIAL ===
Filas:  15
Nulos:
ciudad          2
departamento    2
salario         2
fecha_ingreso   2
activo          1

Duplicados: 2

Tipos:
salario    object
activo     object

=== ESTADO FINAL ===
Filas:  10
Nulos:
ciudad          0
departamento    0
salario         0
fecha_ingreso   2
activo          0

Tipos:
salario    float64
activo     bool

Salario máximo: $52,000
Salario mínimo: $29,000
```

Fijate que `fecha_ingreso` todavía tiene nulos — decidimos no eliminar esas filas porque la fecha no es imprescindible para el análisis salarial. Cada decisión de limpieza depende del análisis que necesitás hacer, no de una regla universal.

---

## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `.isnull()` | Detectar nulos — devuelve `True`/`False` por celda |
| `.isnull().sum()` | Contar nulos por columna |
| `.dropna()` | Eliminar filas (o columnas) que tienen nulos |
| `.dropna(subset=[...])` | Eliminar solo si hay nulo en columnas específicas |
| `.fillna(valor)` | Reemplazar nulos por un valor dado |
| `.duplicated()` | Detectar filas duplicadas — `True` para las repetidas |
| `.drop_duplicates()` | Eliminar filas duplicadas, conservando la primera |
| `.astype(tipo)` | Convertir el tipo de una columna |
| `pd.to_numeric(..., errors="coerce")` | Convertir a número manejando errores sin romper el programa |
| `.map(diccionario)` | Reemplazar valores de una columna según un diccionario |
| `.value_counts()` | Contar cuántas veces aparece cada valor único en una columna |

---

## Recursos adicionales

- [Pandas Docs — Working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html)
- [Pandas Docs — Duplicated data](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.drop_duplicates.html)
- [Pandas Docs — dtypes](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html)
- [Real Python — Cleaning Data with Pandas](https://realpython.com/python-data-cleaning-numpy-pandas/)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 08 — Exploración Inicial y Visualización](../clase-08/README.md) · [Módulo 2](../README.md) · Clase 10 — Análisis Exploratorio Avanzado →*