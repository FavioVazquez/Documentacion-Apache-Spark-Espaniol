Este turorial provee una introducción rápida al uso de Spark. Introduciremos prinero la consola [shell] intectiva (en Python o Scala), luego mostraremos como escribir aplicaciones en Java, Scala y Python. Para una referencia más completa ver la
[guía de programación](http://spark.apache.org/docs/latest/programming-guide.html) (este link redirije a la documentación en inglés por los momentos).

Para seguir esta guía, primero descarga una versión empaquetada
de Spark de la página [web de spark](http://spark.apache.org/downloads.html). Debido a que no utilizaremos HDFS, puedes descargar un paquete para cualquier
versión de Hadoop.

# Análisis interactivo con la consola de Spark

## Lo Básico

La consola de Spark provee una manera simple de aprender la API,
así como una herramienta poderosa para analiar datos interactivamente. Está disponible tanto en Scala (que corre en la VM de Java y es entonces una manera por tanto es una buena manera de usar librerías existentes de Java) o Python. Arráncala corriendo lo siguiente en el directorio de Spark:

#### En Scala:

```
./bin/spark/shell
```

##### En Python:

```
./bin/pyspark
```

La abstracción principal en Spark es una colección de elementos distribuida llamada un "Conkunto de datos distribuidos resiliente" [Resilient Distributed Dataset] (RDD). Los RDDs pueden ser creados desde InputFormats de Hadoop (como archivos HDFS) o transformando otros RDDs. Hagámos un nuevo RDD desde el
texto del archivo README en el directorio fuente de Spark:

#### En scala:

```
scala> val textfile = sc.textfile("readme.md")
textfile: spark.rdd[string] = spark.mappedrdd@2ee9b6e3
```

##### En Python:

```
>>> textFile = sc.textFile("README.md")
```

Los RDDs tienen [acciones](http://spark.apache.org/docs/latest/programming-guide.html#actions), que retornan valores, y [transformaciones](http://spark.apache.org/docs/latest/programming-guide.html#transformations), que retornan apuntadores a nuevos RDDs. Comencemos con unsas pocas acciones:


#### En Scala:

```
scala> textFile.count() // Number of items in this RDD
res0: Long = 126

scala> textFile.first() // First item in this RDD
res1: String = # Apache Spark
```

#### En Python:

```
>>> textFile.count() # Number of items in this RDD
126

>>> textFile.first() # First item in this RDD
u'# Apache Spark'
```

Usemos ahora una transformación. usaremos la transformación [filter](http://spark.apache.org/docs/latest/programming-guide.html#transformatio), para devolver un nuevo RDD con un subconjunto de los elementos en el archivo:


#### En Scala:

```
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark: spark.RDD[String] = spark.FilteredRDD@7dd4af09
```

#### En Python:

```
>>> linesWithSpark = textFile.filter(lambda line: "Spark" in line)
```

Podemos encadenar estas transformaciones y acciones:

#### En Scala:

```
scala> textFile.filter(line => line.contains("Spark")).count() // How many lines contain "Spark"?
res3: Long = 15

```
#### En Python:

```
>>> textFile.filter(lambda line: "Spark" in line).count() # How many lines contain "Spark"?
15
```
