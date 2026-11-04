# Diagnóstico de calidad de datos: criminalidad en Medellín

Diagnóstico inicial de calidad de los datasets del proyecto **"Análisis de los delitos más frecuentes en Medellín y su variación a través del tiempo"**, desarrollado con Python y Pandas.

**Evidencia:** GA1-220501094-01-AA1-EV06 · SENA
**Aprendiz:** Liliana Sofia Panesso Mendez

## Propósito

Determinar si los datasets seleccionados para el proyecto presentan condiciones adecuadas para su análisis, identificando su estructura, tipos de variables, valores faltantes y posibles inconsistencias, y proponer las acciones de limpieza y preparación necesarias.

## Pregunta analítica

¿Cuáles son los delitos con mayor cantidad de casos registrados en Medellín y cómo ha variado su número de casos entre 2020 y 2025?

## Datasets

| Archivo en `data/raw/` | Fuente | Descarga |
|---|---|---|
| `criminalidad_anio_mes.csv` | MEData, Alcaldía de Medellín (Secretaría de Seguridad y Convivencia). Licencia CC BY-SA 4.0 | [CSV en MEData](http://medata.app.medellin.gov.co/sites/default/files/distribution/1-027-23-000306/consolidado_cantidad_casos_criminalidad_por_anio_mes.csv) · [Ficha en Datos Abiertos Colombia](https://www.datos.gov.co/dataset/Consolidado-cantidad-de-casos-criminalidad-por-a-o/ha7m-7wwj) |
| `criminalidad_comunas_anio.csv` | MEData, Alcaldía de Medellín (Secretaría de Seguridad y Convivencia). Licencia CC BY-SA 4.0 | [CSV en MEData](http://medata.app.medellin.gov.co/sites/default/files/distribution/1-027-23-000303/consolidado_cantidad_casos_criminalidad_en_comunas_por_anio.csv) · [Ficha en Datos Abiertos Colombia](https://www.datos.gov.co/dataset/Consolidado-cantidad-de-casos-criminalidad-en-comu/wtnd-h3vi) |
| `homicidios_banco_mundial.csv` | Banco Mundial, indicador VC.IHR.PSRC.P5 (homicidios intencionales por cada 100.000 habitantes). Archivo original: `API_VC.IHR.PSRC.P5_DS2_es_csv_v2_13913.csv` | [Descarga por API (ZIP)](https://api.worldbank.org/v2/es/indicator/VC.IHR.PSRC.P5?downloadformat=csv) · [Página del indicador](https://datos.bancomundial.org/indicador/vc.ihr.psrc.p5) |

Los archivos se incluyen en el repositorio sin modificaciones porque son datos abiertos y de tamaño reducido (menos de 150 KB cada uno).

## Estructura del repositorio

```
criminalidad-medellin/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/                          # datasets originales, sin modificar
├── notebooks/
│   └── diagnostico_calidad.ipynb     # carga, inspección y diagnóstico
└── reports/
    └── informe_diagnostico.md        # informe técnico con hallazgos y conclusiones
```

## Requisitos

- Python 3.12 o superior
- Git

Las dependencias están en `requirements.txt` (pandas 3.1.1 y JupyterLab 4.6.5, entre otras).

## Cómo reproducir el diagnóstico

Comandos para Windows PowerShell:

```powershell
git clone https://github.com/TU-USUARIO/criminalidad-medellin.git
cd criminalidad-medellin
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
jupyter lab
```

En macOS o Linux, el entorno se activa con `source .venv/bin/activate`.

En JupyterLab, abre `notebooks/diagnostico_calidad.ipynb` y ejecuta **Run → Restart Kernel and Run All Cells**.

**Nota sobre la carga:** el archivo del Banco Mundial se carga con `skiprows=4`, porque sus cuatro primeras líneas son metadatos. Los archivos de MEData usan coma como separador y codificación UTF-8.

## Resultados principales

- Los datasets de Medellín no tienen valores nulos ni duplicados, y sus totales anuales coinciden entre sí.
- Los tres datasets terminan en 2023: no hay datos de 2024 ni de 2025.
- `Codigo_comuna` contiene `SIN DATO` (3,4 % de los casos de 2020–2023) y códigos escritos como entero y como decimal.
- `Fecha_hecho` tiene distinto tipo y significado en los dos datasets de Medellín.

El detalle se encuentra en el [informe técnico](reports/informe_diagnostico.md).