# Informe de calidad de datos – Instacart (EV04)

**Actividad:** GA1-220501094-01-AA1-EV04 – Fundamentos en Pandas
**Aprendiz:** Liliana Sofia Panesso Mendez
**Notebook de origen:** [`notebooks/data_quality.ipynb`](../notebooks/data_quality.ipynb)

Todas las cifras de este informe provienen de las salidas de ese notebook.

## 1. Problema

Instacart tiene muchos datos guardados, pero no sabe con claridad cómo compran sus clientes (a qué horas piden, cada cuánto vuelven y si repiten productos), ni conoce la calidad de esos datos. El objetivo de la actividad fue cargar, revisar y consultar los cinco archivos con Python y Pandas, describir pedidos y productos, detectar limitaciones y responder tres preguntas:

- **P1.** ¿En qué horas del día se concentran los pedidos?
- **P2.** ¿Cada cuánto tiempo vuelven a comprar los clientes?
- **P3.** ¿Qué tanto repiten los clientes los mismos productos?

## 2. Archivos

| Archivo | Qué representa cada fila | Filas | Columnas |
|---|---|---|---|
| `instacart_orders.csv` | Un pedido hecho por un usuario | 478 967 | 6 |
| `products.csv` | Un producto del catálogo | 49 694 | 4 |
| `aisles.csv` | Un pasillo | 134 | 2 |
| `departments.csv` | Un departamento | 21 | 2 |
| `order_products.csv` | Un producto dentro de un pedido | 4 545 007 | 4 |

Los archivos usan punto y coma como separador (`sep=";"`). Se relacionan por `order_id` (orders ↔ order_products), `product_id` (products ↔ order_products), `aisle_id` (products ↔ aisles) y `department_id` (products ↔ departments). `user_id` identifica a los usuarios y solo está en `orders`.

## 3. Diagnóstico de calidad

| Archivo | Valores ausentes | Duplicados | Observaciones |
|---|---|---|---|
| orders | 28 819 en `days_since_prior_order` | 15 filas | Todos los ausentes corresponden a `order_number == 1` (primer pedido: no hay pedido anterior). La columna es `float64` por los `NaN`. Las filas duplicadas tienen `order_dow` = 3 y `order_hour_of_day` = 2. |
| products | 1 258 en `product_name` | 0 | Todos están en el pasillo 100 y el departamento 21, llamados "missing". |
| aisles | 0 | 0 | — |
| departments | 0 | 0 | — |
| order_products | 836 en `add_to_cart_order` | 0 | La columna es `float64` por los `NaN`. |

**Tipos de dato:** las columnas de identificadores y conteos son `int64`. `days_since_prior_order` y `add_to_cart_order` son `float64` porque tienen `NaN`. Las columnas de texto aparecen como `str` en pandas 3 (antes `object`).

**Otras observaciones:**
- Hay 478 952 pedidos únicos frente a 478 967 registros: la diferencia son los 15 duplicados.
- 28 906 registros de `orders` no tienen productos en `order_products`.
- Los productos en la posición 1 del carrito (450 046) coinciden con los pedidos distintos de `order_products` (450 046).

## 4. Hallazgos

- **P1 – Horas:** la hora con más registros de pedidos es las 10 (40 578). Entre las 9 y las 16 h se concentra el 64,95 % (311 079 registros). En la madrugada (0 a 5 h) solo hay el 1,85 % (8 853).
- **P2 – Frecuencia:** el promedio de días entre pedidos es 11,10 (mediana 7), calculado con 450 148 valores. El 50,37 % de los pedidos con pedido anterior llega en 7 días o menos. Los valores más frecuentes son 30 días (51 338) y 7 días (44 579).
- **P3 – Recompra:** el 59,05 % de los registros de `order_products` (2 683 838) son productos que el cliente ya había comprado, y el 87,95 % de los pedidos (395 819 de 450 046) incluye al menos uno.
- **Usuarios:** 157 437 usuarios distintos, con 3,04 pedidos por usuario en el archivo. El promedio de `order_number` es 17,16 (mediana 11).
- **Día de la semana:** los días 0 (84 090) y 1 (82 185) tienen más pedidos.
- **Catálogo:** el departamento 1 (frozen) tiene 4 008 productos (8,07 %).

## 5. Limitaciones

1. Valores ausentes en tres columnas (28 819, 1 258 y 836).
2. No hay fechas reales: solo día de la semana, hora y días desde el pedido anterior.
3. `order_dow` no trae un diccionario que confirme qué día es el 0.
4. 30 es a la vez el valor máximo y el más frecuente de `days_since_prior_order`: posiblemente sea un tope.
5. La información está repartida en varias tablas.
6. `order_products` tiene 4 545 007 filas, lo que hace las operaciones más pesadas.
7. Hay 15 filas duplicadas y 28 906 registros de pedidos sin productos.
8. Es una muestra parcial: 3,04 pedidos por usuario frente a un `order_number` promedio de 17,16.

## 6. Conclusiones

1. Los clientes compran de día, sobre todo a media mañana: el 64,95 % de los registros de pedidos se concentra entre las 9 y las 16 h.
2. La recompra es la norma: el 59,05 % de los productos comprados ya se habían pedido antes y el 87,95 % de los pedidos incluye al menos un producto repetido.
3. Hay un ritmo semanal marcado: la mitad de los pedidos con pedido anterior llega en 7 días o menos, junto a un grupo que vuelve a los 30 días.

Estos problemas de calidad (duplicados, ausentes, nombres faltantes y tipos `float64`) son el punto de partida de la limpieza de la actividad EV06 (ver [`processing_report.md`](processing_report.md)).
