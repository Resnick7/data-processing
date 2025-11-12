# 🧾 **Proceso ETL y análisis de datos de videojuegos en Databricks**

## **Introducción**

El presente trabajo práctico tiene como objetivo aplicar los conceptos fundamentales del procesamiento de datos masivos utilizando **Apache Spark** dentro del entorno **Databricks Community Edition**.
Se desarrolla un flujo **ETL (Extracción, Transformación y Carga)** sobre un dataset público de ventas de videojuegos, con el propósito de explorar, limpiar y analizar los datos de manera distribuida, y finalmente almacenar los resultados en formato **Delta Lake**, un estándar de almacenamiento transaccional optimizado para entornos de big data.

El proyecto permite familiarizarse con el uso de **notebooks, clusters y operaciones con PySpark**, reforzando el vínculo entre la teoría del procesamiento distribuido y la práctica aplicada a un contexto real del mercado laboral relacionado con la **industria de los videojuegos**.


## **Configuración del entorno**

Para el desarrollo se utilizó **Databricks Community Edition**, una plataforma en la nube que integra Apache Spark y ofrece un entorno interactivo para ejecutar código en **Python, SQL, Scala o R**.

Las principales configuraciones fueron:

* Creación de un **cluster** basado en Apache Spark.
* Configuración de un **notebook** asociado al cluster.
* Importación y lectura de datos mediante la librería PySpark.
* Exploración inicial mediante comandos como `df.show()` y `df.printSchema()` para verificar la estructura y tipos de datos.

Esta configuración permite aprovechar la **computación distribuida**, en la que las tareas se paralelizan en varios nodos del cluster, incrementando la eficiencia y escalabilidad.

## **Ingesta de datos**

El dataset seleccionado fue **“Video Game Sales” (vgsales.csv)**, que contiene información sobre más de 16.000 títulos, incluyendo su plataforma, género, desarrollador y ventas por región.
El archivo se obtuvo desde un repositorio público de GitHub y se cargó al entorno de trabajo de Databricks mediante el comando:

```python
df = spark.read.option("header", True).csv("/Volumes/workspace/default/tmp_steam/vgsales.csv")
```

Luego, se visualizaron las primeras filas con:

```python
df.show(5)
```

y se inspeccionó la estructura con:

```python
df.printSchema()
```

Esta etapa permitió confirmar la correcta lectura del archivo y el reconocimiento de los tipos de datos (string, double, etc.).
En caso de que las columnas numéricas fueran importadas como texto, se aplicaron conversiones de tipo con `cast()` para asegurar la correcta manipulación analítica posterior.


## **Transformaciones y limpieza de datos**

Durante esta fase se aplicaron transformaciones sobre el DataFrame original con los siguientes objetivos:

* **Eliminación de duplicados:**

  ```python
  df = df.dropDuplicates()
  ```

* **Manejo de valores nulos:**
  Se eliminaron registros con datos faltantes en campos clave como *Name* o *Global_Sales*:

  ```python
  df = df.na.drop(subset=["Name", "Global_Sales"])
  ```

* **Creación de columnas derivadas:**
  Por ejemplo, se creó una nueva columna con el total de ventas globales en millones:

  ```python
  df = df.withColumn("Global_Sales_Millions", df["Global_Sales"].cast("double") * 1.0)
  ```

* **Filtrado de datos:**
  Se filtraron los títulos con ventas globales mayores a 1 millón para focalizar el análisis en los juegos más exitosos:

  ```python
  df_filtered = df.filter(df["Global_Sales_Millions"] > 1)
  ```

* **Agregaciones:**
  Se realizaron agrupaciones por género y plataforma para calcular promedios de ventas:

  ```python
  df_grouped = df.groupBy("Genre").agg({"Global_Sales_Millions": "avg"})
  ```

Estas transformaciones permitieron obtener un dataset limpio y coherente, apto para análisis descriptivos y visualizaciones posteriores.

## **Almacenamiento en Delta Lake**

Una vez obtenido el conjunto de datos transformado, se guardó en formato **Delta Lake**, que extiende el formato Parquet incorporando capacidades ACID (Atomicidad, Consistencia, Aislamiento y Durabilidad), control de versiones y lecturas/escrituras incrementales.

El proceso se realizó con:

```python
df.write.format("delta").mode("overwrite").save("/Volumes/workspace/default/tmp_steam/vgsales_clean_delta")
```

Posteriormente, se leyó nuevamente desde Delta para validar su integridad:

```python
df_delta = spark.read.format("delta").load("/Volumes/workspace/default/tmp_steam/vgsales_clean_delta")
df_delta.show(5)
```

El uso de Delta Lake garantiza **eficiencia en las consultas**, **consistencia en los datos** y la posibilidad de realizar operaciones de **time travel** (recuperación de versiones anteriores del dataset), características fundamentales en entornos empresariales modernos.

## **Visualización y análisis exploratorio**

En esta etapa se realizaron visualizaciones básicas dentro del notebook, con el objetivo de identificar patrones relevantes.
Por ejemplo:

* Ventas promedio por género de videojuego.
* Distribución de títulos por plataforma.
* Tendencia de lanzamientos por año.

Las visualizaciones se generaron mediante funciones integradas de Databricks (`display(df_grouped)`) o con librerías externas como **matplotlib** o **seaborn** en Python.
Estas representaciones gráficas permiten comunicar de forma clara los resultados del análisis.

## **Conclusiones**

El desarrollo de este trabajo permitió aplicar de manera práctica los conceptos fundamentales de procesamiento de datos en la nube:

* **Databricks** se consolidó como una herramienta potente y accesible para la gestión de grandes volúmenes de información.
* **Apache Spark** demostró su eficiencia en la lectura, transformación y agregación de datos distribuidos.
* **Delta Lake** añadió una capa de confiabilidad y rendimiento que lo hace ideal para arquitecturas **Data Lakehouse** modernas.

Desde una perspectiva temática, el uso de datos del mercado de videojuegos permitió generar un contexto atractivo y actual, con aplicaciones potenciales en áreas de **análisis de mercado**, **predicción de ventas** o **recomendación de productos**.
Este tipo de proyectos integra habilidades muy demandadas en el mercado laboral, como el dominio de entornos de big data, la manipulación de datos con PySpark y la comprensión de arquitecturas de datos modernas.
