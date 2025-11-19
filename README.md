# WHEATHERLAPSE

¿Qué Hace el Código?
Este notebook realiza un análisis avanzado de series de tiempo de datos climáticos para la región de Colombia, con una prueba en un punto particular en la ciudad de Cartagena.

El objetivo principal es:

Procesar y Filtrar un conjunto de datos climáticos de gran volumen (formato NetCDF, probablemente reanálisis ERA5).

Extraer datos para las principales regiones geográficas de Colombia ("Caribe", "Andina", "Amazonía", etc.).

Convertir y Optimizar los datos filtrados en formato Parquet utilizando la librería Modin (basada en Dask) para un manejo eficiente de grandes volúmenes de datos.

Realizar Análisis de Series de Tiempo (Descomposición Estacional, Identificación de Tendencias) para un punto de grilla específico cercano a Cartagena.

Generar un dashboard interactivo usando Panel y hvplot para visualizar dinámicamente las variables climáticas a lo largo del tiempo.

Requisitos Previos
Para ejecutar este notebook correctamente, necesitará un entorno de Jupyter (como Google Colab o un entorno local de Anaconda) con conexión a internet para descargar el archivo de datos.

Instalación de Dependencias
El notebook comienza con celdas dedicadas a instalar todas las librerías necesarias. Es fundamental ejecutar estas celdas primero y en el orden especificado para resolver correctamente las dependencias, especialmente las relacionadas con la visualización geográfica (cartopy) y el procesamiento de datos (linearmodels, modin, xarray, netcdf4).

Las librerías instaladas incluyen:

Librerías del Sistema: libproj-dev, libgdal-dev (necesarias para cartopy).

Librerías Python Clave: numpy, linearmodels, shapely, cartopy, xarray, netcdf4, jupyter_bokeh, gdown, modin, panel, hvplot, statsmodels, regionmask.

Flujo de Ejecución Básico
Siga estos pasos, ejecutando las celdas de forma secuencial:

Ejecutar celdas de instalación: Instale las librerías del sistema y de Python (Secciones 1 y 2).

Descarga y Carga de Datos: La celda siguiente descarga el archivo NetCDF (colombia_climate.nc) de gran tamaño (~1.23 GB) desde Google Drive y establece las variables de configuración.

Filtrado y Mapeo Regional: Las celdas subsiguientes definen las cajas delimitadoras de las regiones de Colombia y crean un mapa mostrando la cobertura de los puntos de grilla para cada región.

Generación de Parquet Regional: Se itera sobre cada región para filtrar los datos, limpiar duplicados, y guardar el resultado como un archivo Parquet optimizado (ej. Caribe.parquet).

Extracción de Serie de Tiempo de Cartagena: La función obtener_serie_tiempo_punto se usa para extraer la serie de tiempo completa del punto de grilla más cercano a las coordenadas de Cartagena (10.4 N, -75.5 W).

Visualización y Análisis: Se generan gráficos de diagnóstico (ubicación del punto, gráficos de series de tiempo por variable).

Dashboard Interactivo: La celda final que genera dashboard_final debe ser ejecutada para interactuar con la herramienta de visualización.

Uso del Dashboard interactivo
El dashboard, construido con Panel, ofrece una manera sencilla de explorar los datos de Cartagena:

Seleccionar Variable: Utilice el widget Select para cambiar entre variables climáticas como tp (precipitación total), ssrd (radiación solar superficial), t2m (temperatura a 2m), msl (presión a nivel medio del mar), etc..

Seleccionar Rango de Fechas: Utilice el widget DatetimeRangePicker para acortar el periodo de análisis de la serie de tiempo.

Cómo Funciona el código 
Se sigue un flujo de datos de Big Data a Small Data:

NetCDF (Big Data): Archivo original de grilla espacio-temporal de todo Colombia.

Dataset Regional (Medium Data): Segmentos del NetCDF filtrados por regiones. Se utiliza xarray para el manejo de los datos multidimensionales y regionmask para el enmascaramiento geográfico.

Parquet Regional (Optimized Medium Data): Los datasets regionales se convierten a DataFrames de modin.pandas (que utiliza Dask para paralelizar) y se guardan como Parquet, un formato de columna eficiente.

Pandas/Modin DataFrame de Punto (Small Data): La serie de tiempo para el único punto de grilla más cercano a Cartagena se carga en un DataFrame de modin.pandas para el análisis detallado.

Variables Climáticas y Unidades
trabaja con variables que provienen típicamente del modelo ERA5 (Reanálisis Europeo), con una resolución temporal horaria:

t2m:Temperatura a 2 metros, "Kelvin (K), convertida a menudo a Celsius (°C)"
tp: Precipitación total, Metros (m) o milímetros (mm)
ssrd: Radiación solar de onda corta superficial descendente, Julios por metro cuadrado (J/m2)
msl: Presión a nivel medio del mar, Pascales (Pa) o hectopascales (hPa)

Funciones y Métodos Clave
obtener_serie_tiempo_punto (dataframe, lat_objetivo, lon_objetivo)
Descripción: Localiza el punto de grilla más cercano a las coordenadas lat_objetivo/lon_objetivo mediante el cálculo de la distancia cuadrática.
Retorno: Un DataFrame que contiene todos los registros de la serie de tiempo para ese único punto de grilla cercano.

Descomposición Estacional
Método: Se usa seasonal_decompose de statsmodels.tsa.seasonal.
Parámetro Clave: Se especifica un periodo de 24 horas, lo que implica que el análisis está buscando un patrón de ciclo diario (diurno/nocturno) en las series de tiempo (ej. temperatura, presión).
Componentes: El resultado separa la serie en:Tendencia (trend): La dirección a largo plazo de la serie.Estacionalidad (seasonal): El patrón repetitivo de 24 horas.
Residuo (resid): La parte no explicada por la tendencia o la estacionalidad.

Análisis de Tendencia (Regresión OLS)
Propósito: Se utiliza un modelo de Mínimos Cuadrados Ordinarios (OLS) de statsmodels para cuantificar la tendencia a largo plazo de la temperatura t2m usando la columna trend obtenida de la descomposición estacional como variable independiente.
Fórmula: $T_{2m} \sim \beta_0 + \beta_1 \times \text{Tendencia} + \epsilon$

Visualización
Mapeo: Utiliza cartopy para la proyección cartográfica y cfeature (Natural Earth) para elementos geográficos (costas, fronteras).
Interactivo: Utiliza hvplot para generar gráficos dinámicos que se integran en el panel de control interactivo de Panel.
