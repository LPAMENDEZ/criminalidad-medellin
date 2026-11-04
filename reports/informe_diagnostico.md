# Informe técnico: diagnóstico inicial de calidad de datos

**Evidencia:** GA1-220501094-01-AA1-EV06
**Aprendiz:** Liliana Sofia Panesso Mendez
**Programa:** Análisis y Desarrollo de Software, SENA
**Fecha:** 4 de noviembre de 2026
**Notebook de soporte:** [`notebooks/diagnostico_calidad.ipynb`](../notebooks/diagnostico_calidad.ipynb)

---

## 1. Nombre del proyecto y pregunta analítica

**Proyecto:** Análisis de los delitos más frecuentes en Medellín y su variación a través del tiempo.

**Pregunta analítica:** ¿Cuáles son los delitos con mayor cantidad de casos registrados en Medellín y cómo ha variado su número de casos entre 2020 y 2025?

El propósito de este informe es determinar si los datasets seleccionados presentan condiciones adecuadas para responder esa pregunta y qué acciones de preparación o limpieza deben realizarse antes del análisis.

## 2. Descripción de los datasets utilizados

| # | Archivo | Fuente | Contenido | Rol |
|---|---|---|---|---|
| 1 | `criminalidad_anio_mes.csv` | MEData, Alcaldía de Medellín (Secretaría de Seguridad y Convivencia), republicado en Datos Abiertos Colombia. Licencia CC BY-SA 4.0 | Cantidad de casos de 7 conductas delictivas por año y mes | Principal |
| 2 | `criminalidad_comunas_anio.csv` | MEData, Alcaldía de Medellín (Secretaría de Seguridad y Convivencia), republicado en Datos Abiertos Colombia. Licencia CC BY-SA 4.0 | Cantidad de casos de las mismas 7 conductas por comuna o corregimiento y año | Complementario |
| 3 | `homicidios_banco_mundial.csv` | Banco Mundial, Indicadores del desarrollo mundial (datos de UNODC) | Tasa de homicidios intencionales por cada 100.000 habitantes, por país y región, con una columna por año | Contexto nacional |

Las 7 conductas son: extorsión, homicidio, hurto a persona, hurto de carro, hurto de moto, hurto a residencia y hurto de semoviente.

## 3. Resultados de la inspección estructural

Se aplicaron a cada dataset los comandos `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')`.

| Dataset | Filas | Columnas | Tipos de datos | Cobertura |
|---|---|---|---|---|
| `crim_mes` | 1586 | 3 | `Fecha_hecho`: str · `Conducta`: str · `Cantidad_casos`: int64 | 2003-01 a 2023-11 |
| `crim_comunas` | 2569 | 4 | `Fecha_hecho`: int64 · `Conducta`: str · `Codigo_comuna`: str · `Cantidad_casos`: int64 | 2003 a 2023 |
| `homicidios_bm` | 266 | 71 | 4 columnas str · 67 columnas float64 | Con datos de 1990 a 2023; para Colombia, 2020 a 2023 |

- **`crim_mes`:** cada fila representa una conducta en un mes. `Cantidad_casos` va de 1 a 5069, con media de 328,8 y desviación estándar de 660,3.
- **`crim_comunas`:** cada fila representa una conducta, en una zona, en un año. `Cantidad_casos` va de 1 a 14.195. `Codigo_comuna` tiene 43 valores únicos.
- **`homicidios_bm`:** cada fila representa un país o una región. Se cargó con `skiprows=4` porque las cuatro primeras líneas del archivo son metadatos.

## 4. Valores faltantes, tipos de datos y posibles inconsistencias

### 4.1 Valores faltantes

- `crim_mes` y `crim_comunas` no tienen valores nulos según `isna().sum()`.
- **Faltante disfrazado:** `Codigo_comuna` contiene el texto `SIN DATO` en 76 registros. `info()` no lo detecta como nulo. En 2020–2023 equivale a 5762 de 169.472 casos (3,4 %).
- **Faltantes implícitos:** `crim_mes` tiene 1586 filas de 1757 combinaciones posibles de mes y conducta. Como el valor mínimo de `Cantidad_casos` es 1 y no hay ceros, los meses sin casos parecen no registrarse.
- `homicidios_bm` tiene valores nulos en 68 de sus 71 columnas; 33 columnas están completamente vacías (1960–1989, 2024, 2025 y `Unnamed: 70`).

### 4.2 Tipos de datos

- `Fecha_hecho` es texto (`str`) en `crim_mes` y entero (`int64`) en `crim_comunas`. En ningún caso es de tipo fecha.
- `Codigo_comuna` es texto por la presencia de `SIN DATO` y de códigos decimales, aunque representa un código numérico.
- `Cantidad_casos` es entero en ambos datasets, lo que es coherente con su significado.
- En pandas 3, las columnas de texto aparecen como `str` y no como `object`. Por eso `describe(include='object')` genera una advertencia de compatibilidad (`Pandas4Warning`).

### 4.3 Posibles inconsistencias

- **Códigos de zona en dos formatos:** los 21 códigos (16 comunas y 5 corregimientos) aparecen como entero (`1`) y como decimal (`1.0`). El formato decimal corresponde solo a la conducta Homicidio, en todos los años.
- **`Fecha_hecho` con distinto significado:** año y mes en `crim_mes`, solo año en `crim_comunas`. Comparar ambos datasets exigió filtros diferentes.
- **`homicidios_bm` mezcla países y regiones:** las filas `AFE` y `AFW` son agregados regionales sin nombre.

### 4.4 Revisiones sin problemas

- No hay registros duplicados en `crim_mes` ni en `crim_comunas`, ni como filas idénticas ni como combinaciones repetidas.
- Las 7 conductas están escritas igual en ambos datasets.
- Los totales anuales de `crim_mes` y `crim_comunas` coinciden exactamente en 2020–2023: 37.816, 45.021, 49.580 y 37.055 casos.
- `Cantidad_casos` no tiene valores negativos.

## 5. Principales hallazgos de calidad por dataset

**`crim_mes` (principal)**
1. **Cobertura incompleta:** los datos terminan en noviembre de 2023. No hay datos de 2024 ni de 2025, aunque el portal indica una última actualización del 20 de octubre de 2026.
2. 2023 tiene solo 11 meses, por lo que su total (37.055 casos) no es comparable con los años completos.
3. La fecha está almacenada como texto.
4. Los meses sin casos no tienen registro.

**`crim_comunas` (complementario)**
1. Misma limitación de cobertura: termina en 2023.
2. `SIN DATO` representa el 3,4 % de los casos de 2020–2023.
3. Los códigos de zona de Homicidio están en formato decimal y los demás en formato entero.
4. Los totales coinciden con `crim_mes`, lo que confirma la coherencia entre ambos datasets.

**`homicidios_bm` (contexto)**
1. Requiere un procedimiento de carga especial (`skiprows=4`).
2. Tiene formato ancho, 33 columnas vacías y 2 columnas constantes.
3. Para Colombia hay datos de 2020 a 2023 (entre 22,6 y 25,6 homicidios por cada 100.000 habitantes); faltan 2024 y 2025.
4. Mide una tasa nacional, no cantidad de casos en Medellín, por lo que no es directamente comparable con los otros dos datasets.

**Limitación común:** los datos corresponden a delitos denunciados y registrados, por lo que existe subregistro de los casos no reportados.

## 6. Acciones de limpieza y preparación propuestas

| Prioridad | Acción | Datasets |
|---|---|---|
| Alta | Definir el período de análisis según la cobertura real (ver conclusiones) | Todos |
| Alta | Convertir `Fecha_hecho` a fecha en `crim_mes` y crear una columna `anio` entera en ambos datasets de Medellín | `crim_mes`, `crim_comunas` |
| Alta | Normalizar `Codigo_comuna`: convertir `1.0` en `1`, reemplazar `SIN DATO` por un valor nulo y convertir la columna a entero | `crim_comunas` |
| Media | Completar con 0 las combinaciones de mes y conducta sin registro, documentando el supuesto | `crim_mes` |
| Media | Calcular las estadísticas por conducta y no de forma general | `crim_mes`, `crim_comunas` |
| Media | Filtrar la fila de Colombia, eliminar las columnas vacías y constantes, y transformar a formato largo (año, valor) | `homicidios_bm` |
| Baja | Reportar el porcentaje de casos `SIN DATO` en todo análisis por comuna | `crim_comunas` |

En esta fase no se ejecutaron procesos de limpieza: los archivos de `data/raw/` se conservan sin modificaciones.

## 7. Conclusiones

1. **Los datasets son confiables, pero no cubren todo el período de la pregunta.** Los datos de Medellín no tienen valores nulos ni duplicados, usan las mismas categorías de delito y sus totales coinciden entre sí. Sin embargo, los tres datasets terminan en 2023 y la pregunta analítica abarca hasta 2025.

2. **Para continuar con el proyecto se proponen dos alternativas,** que se evaluarán en la siguiente fase:
   - **Alternativa A:** ajustar el período de análisis a 2020–2022, que son los años completos, y presentar 2023 de forma separada, aclarando que solo tiene 11 meses.
   - **Alternativa B:** conservar el período 2020–2025 e incorporar una fuente complementaria para 2024 y 2025, verificando que use las mismas categorías y la misma forma de medición; o consultar a la Alcaldía de Medellín por una versión actualizada del consolidado.

3. **Los problemas de calidad encontrados son corregibles.** Los formatos de fecha, los códigos de zona en dos formatos y el valor `SIN DATO` pueden tratarse con transformaciones sencillas en Pandas, sin perder información.

4. **El dataset del Banco Mundial solo debe usarse como contexto.** Mide una tasa nacional y no la cantidad de casos en Medellín, por lo que no puede compararse directamente con los otros dos.

5. **La revisión mostró que `info()` no es suficiente para diagnosticar la calidad.** Problemas como `SIN DATO`, los códigos `1` y `1.0` o los meses sin registro solo se detectaron revisando los valores con `value_counts()`, `describe()` y filtros.