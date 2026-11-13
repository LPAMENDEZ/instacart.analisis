# Informe de procesamiento de datos – Instacart (EV06)

**Actividad:** GA1-220501094-01-AA1-EV06 – Limpieza, análisis y estructuración de datos con Pandas (Instacart)
**Aprendiz:** Liliana Sofia Panesso Mendez
**Notebook de origen:** [`notebooks/pandas_labs.ipynb`](../notebooks/pandas_labs.ipynb)

Todas las cifras de este informe provienen de las salidas de ese notebook.

## 1. Objetivo y pregunta orientadora

Continuar el proyecto de la actividad EV04 ([`data_quality.md`](data_quality.md)): limpiar las tablas de Instacart, analizarlas con filtros y agrupaciones, combinarlas en una tabla maestra y guardar los resultados procesados.

**Pregunta orientadora:** ¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

## 2. Datos de entrada

Archivos en `data/raw/`, leídos con `sep=';'`:

| Archivo | Variable | Filas | Columnas |
|---|---|---|---|
| `instacart_orders.csv` | `orders` | 478 967 | 6 |
| `order_products.csv` | `order_prods` | 4 545 007 | 4 |
| `products.csv` | `products` | 49 694 | 4 |
| `aisles.csv` | `aisles` | 134 | 2 |
| `departments.csv` | `departments` | 21 | 2 |

## 3. Decisiones de limpieza (Lección 2)

| Problema | Causa | Decisión | Justificación |
|---|---|---|---|
| 15 filas duplicadas en `orders` (30 filas contando los originales) | Todas tienen día 3 y hora 2: fallo del sistema | **Eliminar** con `drop_duplicates()` | No son pedidos reales; después `order_id` es único (478 952 filas) |
| Columnas enteras leídas como `int64`/`float64` grandes | Tipos por defecto de `read_csv` | **Corregir** con `astype` (`int8`, `int16`, `int32`, `Int16`) | `orders` baja de 23,0 a 7,2 MB y `order_prods` de 145,4 a 54,5 MB |
| 28 817 nulos en `days_since_prior_order` (28 819 antes de quitar duplicados) | Son primeros pedidos: la comparación da `True` | **Conservar** el nulo (tipo `Int16`) | Rellenar con 0 inventaría que el cliente volvió el mismo día y bajaría la media |
| 1 258 nulos en `product_name` | Productos del pasillo 100 y departamento 21 (`missing`) | **Corregir** con `'Unknown'` | El producto existe y se usa en pedidos; no se puede borrar |
| 836 nulos en `add_to_cart_order` | Están en 70 pedidos de 65 productos o más; la posición solo se registró hasta 64 | **Conservar** el nulo (tipo `Int16`) | Son productos comprados; solo falta su posición |
| 403 nombres con espacios dobles y 16 con espacio invisible (`\xa0`) | Texto sin estandarizar | **Corregir** con `.str.replace`, `.str.strip` y `.str.title` | Quedan 0 y 0; cambiaron 6 321 nombres (13,05 % de 48 436) |
| 141 nombres repetidos tras estandarizar (280 productos, 268 de ellos en pedidos) | Duplicados implícitos con `product_id` distinto | **Conservar y documentar** | Borrarlos dejaría pedidos apuntando a productos inexistentes; se agrupa por `product_id` |

### Informe de cambios del pipeline (Laboratorio 2)

| Tabla | Paso | Filas | Nulos | Memoria (MB) |
|---|---|---|---|---|
| orders | 0 crudo | 478 967 | 28 819 | 23,0 |
| orders | 1 sin duplicados | 478 952 | 28 817 | 23,0 |
| orders | 2 tipos | 478 952 | 28 817 | 7,2 |
| products | 0 crudo | 49 694 | 1 258 | 3,1 |
| products | 1 nombres | 49 694 | 0 | 3,1 |
| order_prods | 0 crudo | 4 545 007 | 836 | 145,4 |
| order_prods | 1 tipos | 4 545 007 | 836 | 54,5 |

El pipeline pasó la validación (`Validación superada`) y produce exactamente las mismas tablas que la limpieza manual (`True True`).

**Otras comprobaciones (Reto 2):** `order_prods` no tiene filas duplicadas ni pares `order_id` + `product_id` repetidos, y `reordered` solo vale 0 o 1. `aisles` y `departments` no necesitan limpieza de texto. Los productos `'Unknown'` aparecen en 9 547 filas (0,21 %) de 8 254 pedidos y se conservan. Convertir el día de la semana a categoría ordenada reduce la memoria de 7,14 a 0,48 MB.

## 4. Perfil de hábitos de compra (Lección 3)

- **Hora pico:** las 10 a. m., con 40 578 pedidos. La celda más alta de la matriz hora × día es el lunes a las 10 (7 881).
- **Días:** el día 0 (domingo, según la convención del material) tiene más pedidos (84 090) y la mayor espera media (11,8 días). La combinación más activa es domingo en la tarde (42 340).
- **Segmentos de clientes** (157 437 clientes, mediana de 2 pedidos, máximo 28):

| Segmento | Clientes | % clientes | Pedidos | Espera mediana (días) |
|---|---|---|---|---|
| ocasional (1 pedido) | 55 357 | 35,2 | 55 357 | 16 |
| regular (2 a 4) | 71 553 | 45,4 | 191 649 | 11 |
| frecuente (5 o más) | 30 527 | 19,4 | 231 946 | 6 |

- **Perfil por franja horaria (% de pedidos de cada segmento):**

| Segmento | Madrugada | Mañana | Tarde | Noche |
|---|---|---|---|---|
| ocasional | 2,1 | 30,0 | 49,4 | 18,5 |
| regular | 1,9 | 31,6 | 48,4 | 18,1 |
| frecuente | 1,7 | 35,2 | 45,9 | 17,2 |

- **Reto 3:**
  - Los domingos, las horas con más pedidos son las 15 h (7 585), las 14 h (7 533) y las 13 h (7 483).
  - La espera media es mayor a las 0 h (12,52 días) y menor a las 22 h (10,62 días).
  - 6 327 clientes (4,02 %) tienen 10 pedidos o más.
  - 44 577 pedidos (9,31 % del total) llegan exactamente a los 7 días.
  - 49 420 pedidos tienen 20 productos o más, y el más grande (61355) tiene 127.
  - La tasa de recompra baja con la posición en el carrito: de 0,679 en la posición 1 a 0,617 en la 5.
  - La mediana de espera es de 8 días en fin de semana frente a 7 entre semana.

## 5. Tabla maestra y resultados (Lección 4)

La tabla maestra parte de `order_prods` (tabla de hechos) y le une `products`, `aisles`, `departments` y `orders` con `how='left'` y `validate='m:1'`. El resultado son **4 545 007 filas y 14 columnas**, sin crear ni perder filas. La auditoría con `indicator=True` mostró 450 046 pedidos con productos, 28 906 pedidos sin productos y 0 pedidos de detalle sin pedido.

- **Productos más pedidos:** Banana (66 050), Bag Of Organic Bananas (53 297), Organic Strawberries (37 039), Organic Baby Spinach (33 971) y Organic Hass Avocado (29 773).
- **Departamentos con más artículos:** produce (1 328 302; tasa de recompra 0,651), dairy eggs (759 842; 0,671) y snacks (404 850; 0,574).
- **Tabla por producto (Laboratorio 4):** 45 573 productos pedidos alguna vez, de los cuales 739 tienen al menos 1 000 pedidos. Hay 18 líderes de recompra por departamento; los cinco primeros son:

| Departamento | Producto | Veces | Tasa de recompra |
|---|---|---|---|
| dairy eggs | Whole Organic Omega 3 Milk | 1 286 | 0,863 |
| produce | Banana | 66 050 | 0,844 |
| snacks | Extra Fancy Unsalted Mixed Nuts | 1 436 | 0,822 |
| beverages | Zero Calorie Cola | 1 256 | 0,818 |
| bakery | Organic Honey Sweet Whole Wheat Bread | 1 607 | 0,799 |

- **Reto 4:**
  - Entre los 77 pasillos con al menos 10 000 artículos, las tasas de recompra más altas son milk (0,781), water seltzer sparkling water (0,731) y fresh fruits (0,718); la más baja es spices seasonings (0,152).
  - En la madrugada, beverages entra en el top 3 de departamentos (8,49 %) en lugar de snacks.
  - Los snacks más pedidos son Lightly Salted Baked Snap Pea Crisps (3 462), Original Veggie Straws (2 331) y Sea Salt Pita Chips (2 275).
  - La tabla por cliente tiene 149 626 clientes, con un promedio de 30,38 artículos; el cliente 103634 compró 580.
  - Un `inner` entre pedidos y detalle da 450 046 filas, frente a 478 952 con `left`.
  - El alcohol se pide más el día 5 (3 814 artículos).

## 6. Archivos generados en `data/processed/`

| Archivo | Contenido | Filas | Columnas | Tamaño | ¿Se versiona? |
|---|---|---|---|---|---|
| `instacart_maestra.parquet` | Tabla maestra: un producto dentro de un pedido con su producto, pasillo, departamento y datos del pedido | 4 545 007 | 14 | 103,62 MB | No (supera 50 MB; está en `.gitignore`) |
| `resumen_departamentos.csv` | Artículos y tasa de recompra por departamento | 21 | 2 (+ `department` como índice) | 785 bytes | Sí |
| `ficha_productos.parquet` | Una fila por producto: veces, clientes, tasa de recompra y posición media | 45 573 | 7 | 1,69 MB | Sí |
| `lideres_por_departamento.csv` | Producto con mayor tasa de recompra de cada departamento (mínimo 1 000 pedidos) | 18 | 8 | 1 729 bytes | Sí |

`instacart_maestra.parquet` se regenera ejecutando `notebooks/pandas_labs.ipynb` completo.

## 7. Limitaciones

1. **Pedidos sin productos:** 28 906 pedidos no tienen productos en `order_products`, por lo que no aparecen en la tabla maestra. Por eso la tabla por cliente tiene 149 626 clientes y no 157 437.
2. **Posiciones incompletas en el carrito:** `add_to_cart_order` solo registra hasta la posición 64, y quedan 836 nulos en pedidos grandes.
3. **Productos sin nombre:** 1 258 productos se marcan como `'Unknown'` (pasillo y departamento `missing`).
4. **Duplicados implícitos:** hay 141 nombres de producto repetidos con distinto `product_id`.
5. **Días de la semana:** la interpretación de `order_dow` (0 = domingo) sigue la convención del material; los datos no traen diccionario.
6. **Sin fechas reales:** no se pueden analizar meses ni tendencias.
7. **Muestra parcial:** la mediana es de 2 pedidos por cliente, así que el segmento "ocasional" no implica que el cliente sea nuevo en la tienda.

## 8. Conclusiones

1. **La limpieza era necesaria y quedó documentada:** se eliminaron 15 duplicados, se redujo la memoria de `orders` de 23,0 a 7,2 MB y de `order_prods` de 145,4 a 54,5 MB, y cada nulo se trató según su causa. El pipeline reproduce el resultado (`True True`).
2. **Pocos clientes concentran la actividad:** el 19,4 % de los clientes (frecuentes) hace 231 946 de los 478 952 pedidos, vuelve cada 6 días (mediana) y compra más por la mañana (35,2 %).
3. **La recompra depende del producto:** Banana es el más pedido (66 050) y muy fiel (0,844), pero el líder de fidelidad es la leche Omega 3 (0,863), y pasillos como milk (0,781) se recompran mucho más que spices seasonings (0,152).

## 9. Respuesta a la pregunta orientadora

Para obtener información confiable sobre los hábitos de compra, los datos de Instacart se prepararon en tres etapas:

1. **Limpiar con decisiones justificadas** (Lección 2): eliminar duplicados, corregir tipos, tratar cada nulo según su causa y estandarizar el texto. Todo quedó en un pipeline reproducible con informe de cambios y validación automática.
2. **Analizar con filtros y agrupaciones** (Lección 3): traducir cada pregunta a "métrica por grupo" y usar `groupby`, `pivot_table`, `pd.cut` y `crosstab` para perfilar cuándo y con qué frecuencia compran los clientes.
3. **Estructurar combinando tablas** (Lección 4): construir una tabla maestra validada (`validate`, `indicator`, `assert`), agregarla a otros niveles (producto, cliente, pasillo) y guardar los resultados en `data/processed/`.

Gracias a ese proceso, cada cifra del análisis (por ejemplo, la hora pico de las 10 a. m., el segmento frecuente que hace casi la mitad de los pedidos o la tasa de recompra de 0,844 de Banana) se apoya en datos sin duplicados, con tipos correctos y en uniones que no pierden ni multiplican filas.
