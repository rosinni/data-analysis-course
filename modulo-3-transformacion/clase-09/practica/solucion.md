# Clínica MediCare - Pistas (leer solo si están trabados)

<details>
<summary>Pista 1 — Diagnóstico de turnos</summary>

La columna `costo` tiene al menos dos tipos de problemas: un valor `'N/A'` que no es número, y un valor negativo que no tiene sentido como costo de una consulta médica. Ambos van a causar problemas cuando intenten convertir la columna a número.

</details>

<details>
<summary>Pista 2 — Duplicados en pacientes y médicos</summary>

Algunos duplicados no son exactamente iguales en todos los campos — pueden tener el mismo DNI pero distinta ciudad, o el mismo nombre pero distinto ID. Decidan qué columna(s) definen que dos registros son "el mismo" y usen `drop_duplicates(subset=[...])`.

</details>

<details>
<summary>Pista 3 — Convertir costo a número</summary>

Después de manejar los valores problemáticos, usen `pd.to_numeric(df_turnos_limpio["costo"], errors="coerce")` para la conversión. Los valores que no se puedan convertir quedarán como `NaN`.

</details>

<details>
<summary>Pista 4 — Preguntas de análisis</summary>

Para las preguntas 1, 2 y 4 van a necesitar un JOIN entre `turnos` y `medicos`. Para la pregunta 3, alcanza con consultar `pacientes` directamente. Recuerden que después de limpiar los DataFrames en Python, la base de datos SQLite original sigue igual — las consultas del Paso 5 van sobre los datos originales, así que el JOIN lo hacen en SQL.

</details>

---

→ [Ir al Ejercicio](./notebook.ipynb)