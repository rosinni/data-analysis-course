# Solución — Clase 10: Ejercicio Integrador Netflix

> **Importante**
>
> Esta guía **no contiene las soluciones completas**.
> Su objetivo es ayudarte cuando te quedes bloqueado durante el ejercicio.
>
> Intenta resolver cada apartado por tu cuenta antes de consultar las pistas.


## Parte 1 — Exploración

### Pista 1

Antes de responder cualquier pregunta necesitas conocer el dataset.

Pregúntate:

- ¿Cómo veo las primeras filas?
- ¿Cómo conozco la cantidad de registros?
- ¿Cómo veo el tipo de dato de cada columna?



### Pista 2

Si quieres saber si existen valores faltantes, recuerda que Pandas posee métodos para detectar valores nulos.

Además, puedes contar cuántos tiene cada columna.


## Parte 2 — Agrupaciones

### Ejercicio 1

> ¿Cuántos títulos existen por país?

#### Pista

La respuesta no requiere recorrer el DataFrame.

Piensa:

> "Quiero agrupar todas las filas que tengan el mismo país."

Luego debes calcular una métrica sobre cada grupo.


### Ejercicio 2

> ¿Cuántas películas y cuántas series existen?

#### Pista

La columna `type` solamente posee dos valores posibles.

Agrupa por esa columna.



### Ejercicio 3

> ¿Cuántos títulos fueron lanzados cada año?

#### Pista

La columna que necesitas ya existe en el dataset.

Después del agrupamiento recuerda ordenar los años de menor a mayor.



### Ejercicio 4

> ¿Cuál es la clasificación por edades más frecuente?

#### Pista

Agrupa por la columna correspondiente y cuenta los registros.

Luego ordena el resultado de mayor a menor.

 

### Ejercicio 5

> Obtener varias métricas por tipo de contenido.

#### Pista

No hagas cuatro `groupby()` distintos.

Existe un método que permite calcular varias métricas en una sola operación.

Piensa en:

```
agg(...)
```

 

# Parte 3 — Agrupaciones múltiples

### Ejercicio 6

> Agrupar por país y tipo.

#### Pista

`groupby()` puede recibir más de una columna.

En lugar de escribir una columna, prueba pasar una lista.

 

### Ejercicio 7

> Convertir el resultado en una tabla.

#### Pista

El resultado anterior tiene un índice con varios niveles.

Existe un método que transforma uno de esos niveles en columnas.

No necesitas volver a hacer el `groupby()`.

 

# Parte 4 — Pivot Tables

### Ejercicio 8

#### Pista

Pregúntate:

- ¿Qué columna quiero como filas?
- ¿Cuál quiero como columnas?
- ¿Qué información quiero calcular?

Con esas tres respuestas puedes construir una `pivot_table()`.

 

### Ejercicio 9

#### Pista

La lógica es exactamente la misma.

Solo cambian las columnas utilizadas.

 

### Ejercicio 10

#### Pista

Si una combinación no existe, Pandas coloca:

```
NaN
```

Busca un parámetro dentro de `pivot_table()` que permita reemplazar esos valores automáticamente.

 

# Parte 5 — Visualización

### Ejercicio 11

#### Pista

Puedes crear gráficos directamente desde un DataFrame o una Series.

No es necesario llamar a Matplotlib para dibujar las barras.

Solo lo utilizarás para personalizar el gráfico.

 

### Ejercicio 12

#### Pista

Primero obtén la cantidad de películas y series.

Después grafica ese resultado.

 

### Ejercicio 13

#### Pista

La tabla dinámica que construiste anteriormente ya está lista para ser graficada.

No necesitas volver a calcularla.

 

# Desafío — Calidad de datos

Durante el análisis quizá notes algo extraño.

Por ejemplo:

```
United States
United States, Canada
United States, India
```

¿Tiene sentido que sean tres países distintos?

 

### Pista 1

Observa con atención algunos valores de la columna `country`.

¿Qué tienen en común?

 

### Pista 2

Algunas filas contienen varios países separados por comas.

Tal vez primero necesites convertir ese texto en una lista.

Piensa en los métodos para trabajar con cadenas (`str`).

 

### Pista 3

Una vez que tengas una lista de países, existe un método de Pandas capaz de convertir:

```
["Argentina", "Brasil"]
```

en

| country |
|   -|
| Argentina |
| Brasil |

Investiga el método:

```
explode()
```

 

# Insights

Esta sección no evalúa únicamente código.

También evalúa tu capacidad de interpretar los resultados.

No escribas simplemente:

> "Estados Unidos tiene más títulos."

Eso es un dato.

Piensa mejor:

- ¿Qué significa ese resultado?
- ¿Cómo podría utilizar Netflix esa información?
- ¿Qué decisión podría tomar la empresa?

Recuerda:

> Un **dato** describe lo que ocurrió.

> Un **insight** explica por qué ese dato es importante para tomar una decisión.

 

# Si te bloqueas...

Antes de buscar la solución intenta responder estas preguntas:

- ¿Qué columna quiero analizar?
- ¿Cómo quiero agrupar los datos?
- ¿Qué métrica necesito calcular?
- ¿Quiero una Series o una tabla?
- ¿Necesito reorganizar el resultado?
- ¿Sería más claro mostrarlo con un gráfico?

Muchas veces responder esas preguntas es suficiente para descubrir la solución.