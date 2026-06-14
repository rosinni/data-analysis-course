# Soluciones — Clase 01: Introducción al Data Analysis

> Intenta resolver los ejercicios antes de leer esto. La fricción es parte del aprendizaje.


## Ejercicio 1 — Tu primera variable de análisis

```python
nombre = "María García"
edad = 34
monto = 5420.5
es_premium = True

print("Cliente: " + nombre)
print("Edad: " + str(edad))
print("Última compra: $" + str(monto))
print("Premium: " + str(es_premium))

# Alternativa con f-strings (más limpio):
print(f"Cliente: {nombre}")
print(f"Edad: {edad}")
print(f"Última compra: ${monto}")
print(f"Premium: {es_premium}")
```

**Conceptos clave:** `str`, `int`, `float`, `bool` son los cuatro tipos primitivos básicos. Con f-strings no necesitas convertir a string manualmente — Python lo hace por vos.


## Ejercicio 2 — Etiqueta de producto

```python
nombre_producto = "Notebook"
marca = "Lenovo"
precio = 320000
stock = 8

# Sin casting esto falla: no se puede concatenar int con str
etiqueta = marca + " " + nombre_producto + " | Precio: $" + str(precio) + " | Stock: " + str(stock) + " unidades"
print(etiqueta)

# Con f-string (evita el casting manual):
etiqueta = f"{marca} {nombre_producto} | Precio: ${precio} | Stock: {stock} unidades"
print(etiqueta)
```

**El error esperado:** `TypeError: can only concatenate str (not "int") to str`. Este es uno de los errores más frecuentes al empezar — Python no convierte tipos automáticamente en concatenación.


## Ejercicio 3 — Casting para el análisis

```python
precio_str = "15990"
descuento_str = "0.15"
cantidad_str = "4"
tiene_iva_str = "True"

# Conversiones
precio = int(precio_str)
descuento = float(descuento_str)
cantidad = int(cantidad_str)
tiene_iva = tiene_iva_str == "True"  # ojo: bool("True") siempre da True, incluso bool("False")

# Cálculo
precio_con_descuento = precio * (1 - descuento)

if tiene_iva:
    precio_final = precio_con_descuento * 1.21
else:
    precio_final = precio_con_descuento

total = precio_final * cantidad

print(f"Precio original: ${precio}")
print(f"Precio con descuento: ${precio_con_descuento}")
print(f"Precio final (con IVA): ${precio_final:.2f}")
print(f"Total x{cantidad}: ${total:.2f}")
```

**Trampa importante:** `bool("False")` devuelve `True` porque el string `"False"` no está vacío. Para convertir un string `"True"`/`"False"` a booleano de forma segura, usá la comparación `== "True"`.

---

## Ejercicio 4 — Reporte de ventas básico

```python
fecha = "2024-11-14"
vendedor = "Carlos Méndez"
ventas = [45000, 32000, 67500, 28000, 91000]
producto_estrella = "Auriculares Bluetooth"

total = sum(ventas)
promedio = total / len(ventas)

reporte = f"""Reporte de ventas — {fecha}
Vendedor: {vendedor}
Total ventas: ${total}
Promedio por venta: ${promedio}
Producto estrella: {producto_estrella}"""

print(reporte)
```

**Nota:** `sum()` y `len()` son funciones built-in de Python — no necesitas importar nada. Aunque todavía no hablamos de listas en profundidad, estas dos funciones son suficientes para este contexto.


## Ejercicio 5 — Validar tipos antes de operar

```python
usuarios_activos = "1523"
ingresos_totales = 847320.50
tasa_conversion = "3.7"
nombre_campania = "BlackFriday2024"

# Errores originales:
# - usuarios_activos es str, no se puede dividir
# - tasa_conversion es str, no se puede dividir por 100
# - ingreso_por_usuario es float, no se puede concatenar directamente

# Corrección con comentarios de tipos:
usuarios_activos_int = int(usuarios_activos)          # str -> int
tasa_decimal = float(tasa_conversion) / 100           # str -> float, luego operar

ingreso_por_usuario = ingresos_totales / usuarios_activos_int

resumen = (
    "Campaña " + nombre_campania +
    " | Conversión: " + tasa_conversion + "%" +
    " | Ingreso/usuario: $" + str(round(ingreso_por_usuario, 2))
)
print(resumen)

# Versión más limpia con f-string:
resumen = f"Campaña {nombre_campania} | Conversión: {tasa_conversion}% | Ingreso/usuario: ${ingreso_por_usuario:.2f}"
print(resumen)
```

**Tres errores en el código original:**
1. `847320.50 / "1523"` → `TypeError`: no se puede dividir float por string
2. `"3.7" / 100` → `TypeError`: no se puede dividir string por int
3. `"..." + ingreso_por_usuario` → `TypeError`: no se puede concatenar float en string


## Ejercicio 6 — Generador de IDs

```python
anio = 2024
pais = "AR"
tipo_transaccion = "compra"
numero = 847

# Opción 1: zfill
numero_formateado = str(numero).zfill(6)
transaction_id = str(anio) + "-" + pais + "-" + tipo_transaccion.upper() + "-" + numero_formateado

# Opción 2: f-string con formateo numérico (más elegante)
transaction_id = f"{anio}-{pais}-{tipo_transaccion.upper()}-{numero:06d}"

print(transaction_id)
# Resultado: 2024-AR-COMPRA-000847
```

**`{numero:06d}` explicado:** el `0` indica que rellene con ceros, el `6` es el ancho mínimo, y `d` indica entero decimal. Es equivalente a `str(numero).zfill(6)` pero más conciso dentro de un f-string.



## Ejercicio 7 — Detective de tipos

```python
valor_a = "42"
valor_b = "3.14"
valor_c = "True"
valor_d = ""
valor_e = "N/A"
valor_f = "  150  "
valor_g = "$1,250.50"
valor_h = "01/03/2024"

# Análisis y conversión
a = int(valor_a)           # str -> int: directo
b = float(valor_b)         # str -> float: directo
c = valor_c == "True"      # str -> bool: comparación (no bool())
d = None                   # string vacío -> None (valor ausente)
e = None                   # "N/A" no es convertible -> None o manejar como ausente
f = int(valor_f.strip())   # strip() elimina espacios antes y después
g = float(valor_g.replace("$", "").replace(",", ""))  # limpiar símbolo y separador
h = valor_h                # fecha: queda como string por ahora (usaríamos datetime después)

print(f"a={a} ({type(a).__name__})")
print(f"b={b} ({type(b).__name__})")
print(f"c={c} ({type(c).__name__})")
print(f"d={d} ({type(d).__name__})")
print(f"e={e} ({type(e).__name__})")
print(f"f={f} ({type(f).__name__})")
print(f"g={g} ({type(g).__name__})")
print(f"h={h} ({type(h).__name__})")
```

**Conclusión esperada (ejemplo):**

Los problemas más comunes al leer datos reales son: strings con espacios extras, símbolos de moneda o separadores de miles que impiden el casting, valores ausentes representados como texto ("N/A", "", "null"), booleanos guardados como texto, y fechas que llegan como strings. Pandas automatiza muchas de estas conversiones con `pd.read_csv()` y el parámetro `dtype`, pero internamente hace exactamente lo que hicimos acá. Entender el proceso manual nos da control sobre los casos que pandas no maneja bien por defecto.
```