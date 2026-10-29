# P1. ¿Qué tiempo hizo?

Proyecto individual de **Sistemas de Big Data · UD1. El primer pipeline: obtener, integrar y analizar**.

## 1. Descripción del proyecto

El objetivo de este proyecto es construir un pipeline de datos meteorológicos completo utilizando información diaria de la estación de A Coruña/Alvedro entre el 1 de enero de 2021 y el 31 de diciembre de 2025.

El proyecto incluye la descarga de los datos mediante Python, la inspección del archivo original, la limpieza y transformación de los datos, el análisis estadístico, la creación de gráficos, la comparación de rendimiento entre un bucle y una operación vectorizada, y el almacenamiento y consulta de los datos mediante PostgreSQL.

Se analizan las siguientes variables:

- Temperatura media, mínima y máxima, en °C.
- Humedad relativa, en %.
- Precipitación, en mm.
- Viento medio, en km/h.
- Presión atmosférica, en hPa.

### Estación meteorológica

- **Nombre:** A Coruña / Alvedro (aeropuerto).
- **Indicativo:** 08002.
- **Latitud:** 43,3000° N.
- **Longitud:** -8,3833°.
- **Altitud:** 97 m.
- **Periodo analizado:** 2021–2025.

Estos datos identificativos aparecen en la cabecera del archivo descargado.

## 2. Fuente de los datos y condiciones de uso

Los datos proceden originalmente de **Meteostat**, aunque el archivo utilizado en el proyecto ha sido modificado con fines didácticos para simular una exportación con problemas de calidad.

- **Fuente original:** [Meteostat](https://meteostat.net/).
- **Estación:** A Coruña / Alvedro, indicativo 08002.
- **URL de descarga utilizada en el notebook:**  
  https://github.com/bigdatawirtz/datasets/raw/refs/heads/main/datasets/meteo_alvedro_2021_2025.csv
- **Fecha de descarga:** [Añadir la fecha en la que se descargó el archivo].

El archivo se descarga desde el código mediante la biblioteca `requests` y se guarda en `datos/raw/` sin modificar sus bytes originales.

Para las condiciones de uso, debe tenerse en cuenta tanto la licencia aplicable a los datos originales de Meteostat como las condiciones del repositorio que distribuye el archivo didáctico. Se deben respetar las atribuciones y restricciones que correspondan a cada fuente.

- [Web de Meteostat](https://meteostat.net/)
- [Repositorio del archivo utilizado](https://github.com/bigdatawirtz/datasets)

## 3. Estructura del proyecto

La estructura prevista para el repositorio es la siguiente:

```text
p1-que-tempo-fixo/
├── README.md
├── requirements.txt
├── .gitignore
├── datos/
│   ├── raw/
│   │   └── meteo_alvedro_2021_2025.csv
│   └── clean/
│       └── meteo_alvedro_2021_2025.csv
├── notebooks/
│   └── tempo.ipynb
└── output/
```

- `datos/raw/`: contiene el archivo original descargado, sin modificar.
- `datos/clean/`: contiene los datos después del proceso de limpieza.
- `notebooks/`: contiene el notebook con el código, los análisis y las salidas.
- `output/`: carpeta destinada a los resultados que se quieran exportar.
- `requirements.txt`: recoge las dependencias necesarias.
- `.gitignore`: evita incluir archivos temporales, entornos virtuales y otros elementos que no deben formar parte del repositorio.

## 4. Cómo ejecutar el proyecto

### 4.1. Requisitos previos

Es necesario disponer de Python, Jupyter Notebook y Docker con Docker Compose para completar todas las partes del proyecto.

### 4.2. Instalar las dependencias

Desde la carpeta principal del repositorio, se puede crear un entorno virtual e instalar las bibliotecas necesarias:

```bash
python -m venv .venv
```

Activar el entorno virtual:

**Windows:**

```bash
.venv\Scripts\activate
```

**Linux o macOS:**

```bash
source .venv/bin/activate
```

Instalar las dependencias:

```bash
pip install requests pandas matplotlib sqlalchemy psycopg2-binary jupyter
```

### 4.3. Descargar y procesar los datos

Iniciar Jupyter Notebook:

```bash
jupyter notebook
```

Abrir `notebooks/tempo.ipynb` y ejecutar las celdas en orden, desde el principio hasta el final.

El notebook descarga el archivo mediante `requests`, inspecciona su contenido original, lo lee con pandas y realiza las tareas de limpieza y análisis.

**Importante:** las rutas del notebook son relativas a la carpeta `notebooks/`. Para que funcionen correctamente, debe ejecutarse desde esa ubicación y debe existir la estructura de carpetas esperada.

### 4.4. Iniciar PostgreSQL con Docker

En la carpeta donde se encuentre el archivo `docker-compose.yml`, ejecutar:

```bash
docker compose up -d
```

La configuración utilizada en el proyecto es:

```yaml
services:
  db:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: changeme
      POSTGRES_DB: bd
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data
```

Esta configuración crea una base de datos llamada `bd`, con el usuario `user`, y expone PostgreSQL en el puerto `5432`.

Las credenciales se utilizan únicamente para el entorno local de prácticas y deben cambiarse antes de utilizar una configuración real.

### 4.5. Cargar los datos y ejecutar las consultas SQL

Con PostgreSQL en funcionamiento, ejecutar las celdas correspondientes al apartado de base de datos del notebook.

La conexión se realiza con SQLAlchemy y el controlador `psycopg2`. Los datos limpios se cargan en una tabla llamada `meteo` mediante `DataFrame.to_sql()`.

A continuación, se comprueba el número de filas cargadas y se realizan tres consultas SQL:

1. Temperatura media de cada año.
2. Precipitación mensual media, calculada a partir de los totales de cada mes y año.
3. Temperatura máxima absoluta de cada año.

Los resultados de las consultas se comparan con los cálculos realizados previamente con pandas.

Para detener el contenedor cuando ya no se necesite:

```bash
docker compose down
```

## 5. Decisiones de limpieza de los datos

El archivo original contiene problemas de formato y calidad que se detectaron inspeccionando sus bytes, leyendo las primeras líneas, revisando los tipos de las columnas, calculando estadísticas descriptivas y representando gráficamente las variables.

La siguiente tabla resume los problemas y el tratamiento aplicado en el notebook.

| Problema detectado | Cómo se detectó | Tratamiento aplicado |
|---|---|---|
| Codificación distinta de UTF-8 | Inspección de los bytes del archivo y lectura de la cabecera. | Se utilizó `latin-1` para leer correctamente los caracteres. |
| Líneas de metadatos antes de los datos | Inspección de las primeras líneas del archivo. | Se utilizó `skiprows=6` al leer el CSV. |
| Separador de columnas no estándar | Inspección de las líneas del archivo. | Se indicó `sep=";"` en `read_csv()`. |
| Coma como separador decimal | Los valores numéricos aparecían con coma decimal. | Se indicó `decimal=","`. |
| Punto como separador de miles | Se observó en valores como la presión atmosférica. | Se indicó `thousands="."`. |
| Códigos de ausencia de datos | Revisión de los valores y de las primeras líneas del archivo. | Se indicaron `"-"`, cadena vacía, `"NA"` e `"Ip"` en `na_values`. |
| Nombres de columnas poco manejables | Revisión de las columnas después de la lectura. | Se renombraron con nombres cortos como `tmed`, `tmin`, `tmax`, `prec` y `presion`. |
| Fecha leída inicialmente como texto | Inspección de los tipos con `info()` y `dtypes`. | Se convirtió la columna `data` a fecha mediante `pd.to_datetime()`. |
| Filas duplicadas | Se utilizó `duplicated()`. Se encontraron 8 filas duplicadas exactas. | Se eliminaron las filas completamente duplicadas con `drop_duplicates()`. |
| Fechas repetidas | Se utilizó `meteo["data"].duplicated()`. Se detectaron 45 apariciones de fechas repetidas. | Se eliminaron las filas duplicadas exactas. El código no elimina necesariamente todos los registros con una misma fecha; este caso requiere una comprobación adicional. |
| Valores centinela `-9999` | Estadísticas descriptivas, especialmente mínimos y medias claramente anómalos. | Se trataron parcialmente mediante las reglas que identifican valores negativos imposibles en temperaturas y precipitación. No se aplica una sustitución general de `-9999` en todas las columnas. |
| Valores meteorológicos fuera de rango o inconsistentes | Se construyó una máscara con condiciones sobre temperaturas y precipitación. Se detectaron 15 filas afectadas por al menos una condición. | Se sustituyeron por valores ausentes algunos valores de precipitación negativos, temperaturas inferiores a -273 °C, temperaturas negativas según las reglas del notebook y temperaturas medias superiores a las máximas. |
| Valores ausentes en variables meteorológicas | Se utilizó `isna().sum()` y se revisaron los resultados tras la limpieza. | Se aplicó interpolación temporal a `tmed`, `tmax`, `tmin`, `vento`, `hum` y `presion`. |
| Precipitación ausente | Se revisó el recuento de valores ausentes tras la limpieza. | No se interpoló la precipitación, ya que no es adecuado asumir que los valores entre dos observaciones pueden estimarse linealmente. Quedaron 130 valores ausentes. |
| Posibles días ausentes en la serie | El periodo esperado abarca cinco años y se dispone de una columna de fechas. | Es necesario comprobar explícitamente el calendario completo y las fechas ausentes; el notebook no documenta una reparación de todos los huecos de fechas. |

### Consideraciones sobre la limpieza

La interpolación temporal permite completar huecos en algunas variables continuas, pero introduce estimaciones que no equivalen a mediciones reales. Por ello, no se interpoló la precipitación.

La limpieza también requiere cautela al interpretar los valores anómalos. Una temperatura negativa puede ser meteorológicamente posible; por tanto, las reglas que marcan temperaturas negativas como imposibles deben revisarse y justificarse para evitar eliminar observaciones válidas.

El notebook guarda los datos procesados en `datos/clean/` mediante `to_csv()`. Para que el archivo tenga la extensión convencional, es recomendable guardarlo como `meteo_alvedro_2021_2025.csv`.

## 6. Análisis realizado

El notebook responde a las preguntas del enunciado mediante agregaciones con pandas.

### 6.1. Temperatura media anual

Se calcula la media de `tmed` para cada año. Los resultados obtenidos son:

| Año | Temperatura media |
|---|---:|
| 2021 | 14,1 °C |
| 2022 | 15,0 °C |
| 2023 | 14,9 °C |
| 2024 | 14,5 °C |
| 2025 | 14,4 °C |

Las temperaturas medias anuales no siguen una tendencia ascendente continua: aumentan en 2022 respecto a 2021 y después descienden gradualmente hasta 2025.

### 6.2. Precipitación mensual

Se estudia la distribución mensual de la precipitación y se representa la precipitación mensual media del periodo.

En el cálculo basado en los totales mensuales de cada año, octubre presenta la precipitación media más alta y julio la más baja.

Los resultados de precipitación deben interpretarse teniendo en cuenta que hay datos ausentes que no se han interpolado.

### 6.3. Días de lluvia

Se considera día de lluvia aquel cuya precipitación registrada es **igual o superior a 1 mm**. Se cuentan estos días agrupando por mes.

### 6.4. Días más cálidos y más fríos

Se obtienen los días con la temperatura máxima más alta y la temperatura mínima más baja de cada año mediante agrupaciones de pandas.

## 7. Gráficos

El notebook incluye tres visualizaciones:

1. **Evolución diaria de la temperatura:** representa la temperatura media y su media móvil de 30 días.
2. **Precipitación mensual media:** compara los totales mensuales medios del periodo 2021–2025.
3. **Comparación de temperaturas mensuales:** compara el año 2025 con la media del periodo 2021–2024.

Estas representaciones permiten observar la evolución de las temperaturas, la estacionalidad de las precipitaciones y las diferencias del último año respecto a los anteriores.

## 8. Medición de eficiencia

Se calcula la amplitud térmica diaria como la diferencia entre la temperatura máxima y la mínima:

`amplitud térmica = tmax - tmin`

Se comparan dos implementaciones:

- Un bucle `for` que recorre las filas del DataFrame.
- Una operación vectorizada de pandas que resta directamente las dos columnas.

Las mediciones realizadas con `%timeit` muestran que la operación vectorizada es considerablemente más rápida. En la ejecución registrada, el bucle tardó aproximadamente 49 ms y la operación vectorizada unos 110 μs.

Al repetir la prueba con un DataFrame cinco veces mayor, el bucle tardó aproximadamente 267 ms, mientras que la operación vectorizada tardó unos 101 μs en esa medición.

Estos tiempos dependen del equipo y de las condiciones de ejecución, pero muestran la ventaja de evitar los bucles explícitos en operaciones que pandas puede realizar de forma vectorizada.

## 9. PostgreSQL y consultas SQL

Se utiliza Docker para ejecutar PostgreSQL 17 y SQLAlchemy para conectar Python con la base de datos.

Los datos se cargan en la tabla `meteo` con `to_sql()`. Después se comprueba el número de filas de la tabla y se comparan los resultados de las consultas SQL con sus equivalentes en pandas.

Las tres consultas implementadas corresponden a:

1. Media anual de la temperatura media.
2. Precipitación mensual media calculada a partir de los totales mensuales.
3. Temperatura máxima registrada en cada año.

La comparación permite comprobar que las agregaciones SQL y pandas producen resultados coincidentes cuando aplican los mismos datos y criterios.

## 10. Uso de la inteligencia artificial

Se utilizó **ChatGPT** como herramienta de apoyo durante el desarrollo del proyecto, principalmente para:

- Ayudar a redactar y organizar el archivo `README.md`, siguiendo los requisitos del enunciado.
- Resolver dudas de la parte final de PostgreSQL, especialmente sobre la conexión, la carga de datos y la elaboración de consultas SQL.
- Ayudar a revisar y completar algunas funciones y fragmentos de código de las últimas partes del notebook que no conseguía implementar correctamente.

La IA se utilizó como ayuda para comprender y completar el trabajo, no como sustituto de la comprobación de los resultados. El código y las salidas deben revisarse para verificar que las operaciones realizadas son correctas y que las decisiones de limpieza están justificadas.

## 11. Conclusión

El proyecto permite recorrer las principales etapas de un pipeline de datos: obtención, inspección, limpieza, análisis, visualización, medición de eficiencia y almacenamiento en una base de datos relacional.

El proceso también muestra la importancia de revisar la calidad de los datos antes de analizarlos, ya que los valores ausentes, los duplicados, los códigos centinela y las inconsistencias pueden alterar los resultados estadísticos.
