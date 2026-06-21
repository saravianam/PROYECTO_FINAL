# PROYECTO_FINAL
##1. Título del proyecto 
Proyecto final de EDA y Dashboard de un conjunto de datos obtenido de la red. 

##2. Descripción del proyecto 
Este proyecto analiza la oferta de alojamientos Airbnb en Madrid utilizando datos obtenidos de Inside Airbnb.
El objetivo es identificar los factores que influyen en el precio de los alojamientos, estudiar la distribución geográfica de la oferta y analizar la relación entre características de los anuncios, valoraciones de los huéspedes y precios.
El proyecto incluye procesos de limpieza y transformación de datos con Python, análisis exploratorio y la creación de un dashboard interactivo en Power BI.

##3. Estructura del proyecto
El paso a paso seguido para la realización del proyecto es el siguiente: 
###3.1.CARGA DE DATOS 
####3.1.1.Importación de librerias
import numpy as np
import pandas as pd
####3.1.2. Lectura de .csv y .xlsx 
listado = pd.read_csv("C:/Users/sarav/Documents/Cursos/Data analytics The Power/9. Proyecto Final/listingsMADRID.csv")
calendario = pd.read_excel("C:/Users/sarav/Documents/Cursos/Data analytics The Power/9. Proyecto Final/calendarMADRID.xlsx")

###3.2. EXPLORACIÓN INICIAL DE DATOS
####3.2.1. Número de filas y columnas, tipos de datos, primeras filas…
Archivo LISTADO
Listado.info()
Listado.head()
Listado.shape
Listado[“id”].nunique()

Archivo CALENDARIO
calendario.info()
calendario.head()
calendario.shape
calendario[“listing_id”].nunique()

####3.2.2. Nulos
Archivo LISTADO
listado.isnull()
listado.isnull().sum()
nulos = listado.isnull().sum().sort_values(ascending=False)
nulos.head(20)

Archivo CALENDARIO
calendario.isnull()
calendario.isnull().sum()

####3.2.3. Duplicados
listado.duplicated().sum()
calendario.duplicated().sum()

###3.3. TRANSFORMACION Y LIMPIEZA PROFUNDA 
####3.3.1. Identificación de tipos de datos por columnas
Archivo LISTADO
listado.dtypes.value_counts()
listado.select_dtypes(include="string").columns.tolist()
listado[
[
"last_scraped",
"host_since",
"calendar_last_scraped",
"first_review",
"last_review"
]
].head()

Formato columna Price:
listado["price"].dtype

Valores de porcentajes y booleanos:
listado[
[
"host_response_rate",
"host_acceptance_rate",
"host_is_superhost",
"host_has_profile_pic",
"host_identity_verified",
"instant_bookable"
]
].head()

####3.3.2. Conversión de variables
#####a)De string a fecha
columnas_fecha = [
"last_scraped",
"host_since",
"calendar_last_scraped",
"first_review",
"last_review"
]

for col in columnas_fecha:
listado[col] = pd.to_datetime(listado[col])

listado[columnas_fecha].dtypes

#####b) Price a numero 
listado["price"] = (
listado["price"]
.str.replace("$", "", regex=False)
.str.replace(",", "", regex=False)
.astype(float)
)
listado["price"].describe()

#####c) String a porcentajes
listado["host_response_rate"] = (
listado["host_response_rate"]
.str.replace("%", "", regex=False)
.astype("float")
)

listado["host_acceptance_rate"] = (
listado["host_acceptance_rate"]
.str.replace("%", "", regex=False)
.astype("float")
)

listado[
["host_response_rate", "host_acceptance_rate"]
].describe()

#####d) String a booleanos
columnas_bool = [
"host_is_superhost",
"host_has_profile_pic",
"host_identity_verified",
"instant_bookable"
]

for col in columnas_bool:
listado[col] = listado[col].map({
"t": True,
"f": False
})

listado[columnas_bool].dtypes

for col in [
"host_is_superhost",
"host_has_profile_pic",
"host_identity_verified"
]:
print(f"\n{col}")
print(listado[col].unique())

columnas_bool = [
"host_is_superhost",
"host_has_profile_pic",
"host_identity_verified"
]
for col in columnas_bool:
listado[col] = listado[col].astype("boolean")
listado[columnas_bool].dtypes

####3.3.3. Tratamiento de nulos de gran porcentaje
nulos_pct = (listado.isnull().sum() / len(listado) * 100).sort_values(ascending=False)
nulos_pct.head(15)

listado["calendar_updated"].head()

listado["host_neighbourhood"].value_counts().head(10)

listado["neighbourhood_cleansed"].isnull().sum()

listado["license"].head(10)

listado[
["neighbourhood", "neighbourhood_cleansed"]
].head(10)

listado["neighbourhood"].value_counts(dropna=False).head(10)

listado["neighborhood_overview"].dropna().head(3)

listado[“host_about”].dropna().head(3)

[
col for col in listado.columns
if "url" in col.lower() or "picture" in col.lower()
]

####3.3.4. Eliminación de columnas irrelevantes
listado = listado.drop(columns=["calendar_updated"])
listado = listado.drop(columns=["host_neighbourhood"])
listado = listado.drop(columns=["license"])
listado = listado.drop(columns=["neighbourhood"])
listado = listado.drop(columns=["neighborhood_overview"])
listado = listado.drop(columns=["host_about"])
columnas_url = [
"listing_url",
"picture_url",
"host_url",
"host_thumbnail_url",
"host_picture_url"
]
listado = listado.drop(columns=columnas_url)

####3.3.5. Tratamiento de nulos de menor porcentaje
nulos_pct = (
listado.isnull().sum()
/ len(listado)
* 100
).sort_values(ascending=False)
nulos_pct.head(10)

listado["host_location"].value_counts(dropna=False).head(15)

listado["bathrooms"].describe()
listado["bathrooms"].head(10)

listado["beds"].describe()
listado["beds"].head(10)

listado[
["price", "estimated_revenue_l365d"]
].isnull().sum()
(
listado["price"].isnull()
==
listado["estimated_revenue_l365d"].isnull()
).mean()

listado.loc[
listado["review_scores_rating"].isnull(),
"number_of_reviews"
].describe()

####3.3.6. Tratamiento de nulos del dataframe CALENDAR
calendario.info()
calendario = calendario.drop(columns=["price"])
calendario = calendario.drop(columns=["adjusted_price"])
calendario["available"].value_counts(dropna=False)
calendario["available"] = (
calendario["available"]
.map({"t": True, "f": False})
.astype("boolean")
)

calendario["available"].dtype
calendario.isnull().sum()

####3.3.7. unión de fuentes – Merge 
Se usa el método Merge porque hay una o más columnas comunes (la de ID). Se realizó una unión tipo LEFT JOIN utilizando el dataset de calendario como base, con el objetivo de conservar la totalidad de los registros y enriquecerlos con la información del listado. Además, se tiene en cuenta que la columna de ID en cada dataframe se llama distinto. 
df = calendario.merge(
listado,
left_on="listing_id",
right_on="id",
how="left"
)

dataset.shape

Guardar copia:
dataset.to_csv(
"airbnb_madrid_merged.csv",
index=False
)

Auditoria final:
(
dataset.isnull().sum()
/ len(dataset)
* 100
).sort_values(ascending=False).head(15)

dataset[
[
"price",
"bathrooms",
"beds"
]
].isnull().corr()

dataset_eda = dataset.dropna(
subset=["price", "bathrooms", "beds"]
)
dataset_eda.shape


###3.4. ANALISIS DESCRIPTIVO 
####3.4.1. Análisis 1: Precio 
dataset_eda["price"].describe()
dataset_eda["price"].sort_values(ascending=False).head(20)
dataset_eda.loc[
dataset_eda["price"] == dataset_eda["price"].max(),
 	"id"
].nunique()

alojamientos = dataset_eda.drop_duplicates(subset="id")
alojamientos.shape

####3.4.2. Análisis 2: Barrios más caros
alojamientos = dataset_eda.drop_duplicates(subset="id")
alojamientos.groupby(
"neighbourhood_cleansed"
)["price"].agg(["count","mean"]).sort_values(
"mean",
ascending=False
)

Barrios con 10 alojamientos mínimo:
barrios_precio = (
alojamientos.groupby("neighbourhood_cleansed")["price"]
.agg(["count","mean"])
)
barrios_precio = barrios_precio[barrios_precio["count"] >= 10]
barrios_precio.sort_values("mean", ascending=False)

####3.4.3. Análisis 3: Superhost vs No Superhost
alojamientos.groupby(
"host_is_superhost"
)["price"].agg(["count","mean"])

####3.4.4. Análisis 4: Habitaciones y precio
alojamientos.groupby(
"bedrooms"
)["price"].agg(["count","mean"]).sort_values("mean")

####3.4.5. Análisis 5: Tipo de alojamiento 
alojamientos.groupby(
"property_type"
)["price"].agg(["count","mean"]).sort_values(
"mean",
ascending=False
).head(15)


####3.4.6. Análisis 6: Valoraciones
alojamientos["grupo_valoracion"] = pd.cut(
alojamientos["review_scores_rating"],
bins=[0, 4, 4.5, 4.8, 5],
labels=["Baja", "Media", "Alta", "Excelente"]
)
alojamientos.groupby("grupo_valoracion")["price"].agg(["count","mean"])


###3.5. VISUALIZACION Y DASHBOARD
Las visualizaciones se han realizado mediante Power BI y se encuentran en el archivo correspondiente.  

##4. Instalación y requisitos 
Los datos utilizados proceden de la plataforma Inside Airbnb: https://insideairbnb.com/
Se han utilizado los archivos: Listings (.csv) y Calendar (.xlsx)
Se han utilizado las herramientas: Python 3, Pandas, NumPy, Jupyter Notebook y Power BI

##5. Resultados y conclusiones 
Los resultados se visualizan al ejecutar los códigos.

##6. Contribuciones
Toda sugerencia para mejorar el proyecto es bienvenida.

##7. Autores y Agradecimientos 
Autora: Sara Viana Martínez (https://github.com/saravianam)

