# instacart-analisis

Análisis de datos de compras de Instacart con Python y Pandas, desarrollado como proyecto de formación del SENA.

**Aprendiz:** Liliana Sofia Panesso Mendez

El proyecto parte de cinco archivos CSV de Instacart (pedidos, productos, pasillos, departamentos y productos por pedido) y los explora, limpia, analiza y combina para describir los hábitos de compra de los clientes.

## Actividades

| Actividad | Tema | Notebook | Informe |
|---|---|---|---|
| **EV04** – GA1-220501094-01-AA1-EV04 | Fundamentos en Pandas: carga, exploración, indexación, filtrado y cálculos descriptivos | [`notebooks/data_quality.ipynb`](notebooks/data_quality.ipynb) | [`reports/data_quality.md`](reports/data_quality.md) |
| **EV06** – GA1-220501094-01-AA1-EV06 | Limpieza, análisis y estructuración de datos: lecciones 2, 3 y 4 con sus laboratorios guiados y retos | [`notebooks/pandas_labs.ipynb`](notebooks/pandas_labs.ipynb) | [`reports/processing_report.md`](reports/processing_report.md) |

**Pregunta orientadora de la EV06:** ¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

## Estructura del proyecto

```
instacart-analisis/
├── README.md
├── .gitignore
├── requirements.txt
│
├── data/
│   ├── raw/                  <- CSV originales (no se versionan)
│   │   ├── instacart_orders.csv
│   │   ├── order_products.csv
│   │   ├── products.csv
│   │   ├── aisles.csv
│   │   └── departments.csv
│   └── processed/            <- archivos generados por pandas_labs.ipynb
│
├── notebooks/
│   ├── 00_entorno.ipynb      <- comprobación inicial del entorno
│   ├── data_quality.ipynb    <- EV04: exploración y calidad de datos
│   └── pandas_labs.ipynb     <- EV06: lecciones 2, 3 y 4
│
└── reports/
    ├── data_quality.md       <- resumen de la EV04
    └── processing_report.md  <- informe de procesamiento de la EV06
```

## Dataset

Los cinco archivos CSV de Instacart usan **punto y coma** (`;`) como separador:

| Archivo | Qué representa cada fila | Filas | Columnas |
|---|---|---|---|
| `instacart_orders.csv` | Un pedido de un usuario | 478 967 | 6 |
| `order_products.csv` | Un producto dentro de un pedido | 4 545 007 | 4 |
| `products.csv` | Un producto del catálogo | 49 694 | 4 |
| `aisles.csv` | Un pasillo | 134 | 2 |
| `departments.csv` | Un departamento | 21 | 2 |

**Los CSV no se versionan en Git** por su tamaño (`order_products.csv` pesa unos 90 MB). Para ejecutar el proyecto, descárgalos del material del curso y colócalos en `data/raw/` con exactamente esos nombres.

## Requisitos

- Windows con Python 3.14 (lanzador `py`).
- Las librerías de [`requirements.txt`](requirements.txt), entre ellas pandas 3.1.0, numpy 2.5.4, pyarrow 26.0.1 (para leer y escribir Parquet) y JupyterLab 4.6.4.

## Instrucciones de ejecución (Windows PowerShell)

1. Desde la carpeta del proyecto, crea y activa el entorno virtual:

   ```powershell
   py -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   Si PowerShell bloquea el script de activación, ejecuta una vez `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

2. Instala las dependencias:

   ```powershell
   python -m pip install -r requirements.txt
   ```

3. Coloca los cinco CSV en `data/raw/`.

4. Abre JupyterLab:

   ```powershell
   jupyter lab
   ```

5. Ejecuta los notebooks en este orden, cada uno con **Kernel → Restart Kernel and Run All Cells**:
   1. `notebooks/data_quality.ipynb` (EV04).
   2. `notebooks/pandas_labs.ipynb` (EV06). Debe ejecutarse completo y de arriba hacia abajo, porque cada lección usa las tablas que deja la anterior. Al terminar, crea los archivos de `data/processed/`.

También se pueden ejecutar sin abrir JupyterLab:

```powershell
jupyter nbconvert --to notebook --execute --inplace notebooks/data_quality.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/pandas_labs.ipynb
```

## Archivos procesados (`data/processed/`)

Los genera `notebooks/pandas_labs.ipynb` (Lección 4):

| Archivo | Contenido | Filas × columnas | Tamaño | En Git |
|---|---|---|---|---|
| `instacart_maestra.parquet` | Tabla maestra: cada fila es un producto dentro de un pedido, con nombre, pasillo, departamento y datos del pedido | 4 545 007 × 14 | 103,62 MB | **No** |
| `resumen_departamentos.csv` | Artículos pedidos y tasa de recompra por departamento | 21 × 2 (+ `department`) | 785 bytes | Sí |
| `ficha_productos.parquet` | Una fila por producto: veces pedido, clientes, tasa de recompra y posición media en el carrito | 45 573 × 7 | 1,69 MB | Sí |
| `lideres_por_departamento.csv` | Producto con mayor tasa de recompra de cada departamento (mínimo 1 000 pedidos) | 18 × 8 | 1 729 bytes | Sí |

`instacart_maestra.parquet` supera los 50 MB, por eso está en `.gitignore`. **Para regenerarlo**, ejecuta `notebooks/pandas_labs.ipynb` completo, con los CSV en `data/raw/` y `pyarrow` instalado.

## Flujo de trabajo con Git

- `main`: versiones entregadas.
- `develop`: integración.
- Ramas de trabajo: `feat/...` o `feature/...` (la EV06 se desarrolla en `feature/pandas-data-processing`), que se integran en `develop` con `git merge --no-ff`.
- Las versiones se marcan con tags anotados (`v0.1` a `v0.6`).
