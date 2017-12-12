# Workshop de Big Data con Apache Spark [:)]
# Trabajo Final: M. Jazmín Montero + José Ignacio López Sáez

<!---
All this should be 
commented out
-->

## Infrastructura
El workshop simula una instalacion de produccion utilizando container de Docker.
[docker-compose.yml](docker-compose.yml) contiene las definiciones y configuraciones para esos servicios y sus respectivas UIs:

* Apache Spark: [Spark Master UI](http://localhost:8080) | [Job Progress](http://localhost:4040)
* Apache Kafka:
* Postgres:
* [Superset](http://superset.incubator.apache.org) [Dashboard](http://localhost:8088/)

Los puertos de acceso a cada servicio quedaron los defaults. Ej: spark-master:7077, postgres: 5432

## Levantar ambiente
Instalar [Docker >= 17.03](https://www.docker.com/community-edition).
Correr el script que levanta el ambiente.
**IMPORTANTE** el script `restart-env.sh` borra cualquier dado que haya sido procesado anteriormente.

```bash
./restart-env.sh

# Access Spark-Master and run spark-shell
docker exec -it wksp_master_1 bash
root@588acf96a879:/app# spark-shell
```
Probar:
```scala
val file = sc.textFile("/dataset/DB_Aux_Seminario_Intensivo.txt")
file.count
file.take(10).foreach(println)
```
Acceder a http://localhost:8080 y http://localhost:4040 para ver la SPARK-UI

## Codigo
* Análisis de vuelos en Argentina 2014 a 2017

## Compilar el codigo
Compilar y empaquetar el codigo para deploy en el cluster

```bash
cd code/flights
sbt clean assembly
```

## Submit de un job
Conectarse al Spark-Master y hacer submit del programa

```bash
docker exec -it wksp_master_1 bash

cd /app/us-stock-analysis
spark-submit --master 'spark://master:7077' \
  --class "es.arjon.RunAll" \
  --driver-class-path /app/postgresql-42.1.4.jar \
  target/scala-2.11/flights.jar \
  /dataset/flights /dataset/DB_Seminario_Intensivo.txt /dataset/output.parquet
```
Acceder a http://localhost:8080 y http://localhost:4040 para ver la SPARK-UI

<!---
```## Usando Spark-SQL
Usando SparkSQL para acceder a los datos en Parquet y hacer analysis interactiva. 
```

```bash
docker exec -it wksp_master_1 bash
spark-shell
```

```scala
import spark.implicits._
val df = spark.read.parquet("/dataset/output.parquet")
df.show

df.createOrReplaceTempView("flights")

Usando particiones
val highestClosingPrice = spark.sql("SELECT symbol, MAX(close) AS price FROM stocks WHERE year=2017 AND month=9 GROUP BY symbol")
highestClosingPrice.show
highestClosingPrice.explain

No usando particiones
val highestClosingPrice = spark.sql("SELECT symbol, MAX(close) AS price FROM stocks WHERE full_date > '2017-09-01' GROUP BY symbol")
highestClosingPrice.explain
highestClosingPrice.show
```
-->

## Creando un Dashboard con Superset

* Acceder a http://localhost:8088/, user: `admin`, pass: `superset`.
* Agregar el database (Sources > Databases):
  - Database: `Workshop`
  - SQLAlchemy URI: `postgresql://workshop:w0rkzh0p@postgres/workshop`
  - OK
* Agregar tabla (Sources > Tables) :
  - Database: `workshop`
  - Table Name: `flights`
* Create Slices & Dashboard [official docs](https://superset.incubator.apache.org/tutorial.html#creating-a-slice-and-dashboard)
