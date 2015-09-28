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

Usemos ahora una transformación. usaremos la transformación [filter](http://spark.apache.org/docs/latest/programming-guide.html#transformations), para devolver un nuevo RDD con un subconjunto de los elementos en el archivo:


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

## Más sobre operaciones sobre RDD

#### *Esta sección se hará primero para Scala y luego para Python.*

Las acciones y transformaciones de RDDs pueden ser usadas para
computaciones complejas. Digamos que queremos encontrar la línea con la mayor cantidad de palabras:

#### En Scala:

```
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
res4: Long = 15
```

Esto primero mapea una línea a un valor entero, creando un nuevo RDD.  `reduce` es llamado en ese RDD para encontrar la mayor cuenta en una línea. Los argumentos `map` y `reduce` son literales de función (clousures), y pueden usar cualquier característica o libería de Scala o Java. Por ejemplo, podemos llamar fácilmente funciones declaradas en otra parte. Usaremos
la función `Math.max()` para hacer más fácil de entender este
código:

```
scala> import java.lang.Math
import java.lang.Math

scala> textFile.map(line => line.split(" ").size).reduce((a, b) => Math.max(a, b))
res5: Int = 15
```

Un patrón común para flujo de datos es MapReduce, popularizado
por Hadoop. Spark puede implementar flujos de MapReduce fácilmente:

```
scala> val wordCounts = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a, b) => a + b)
wordCounts: spark.RDD[(String, Int)] = spark.ShuffledAggregatedRDD@71f027b8
```

Aquí, hemos combinado las transformaciones [flatMap](http://spark.apache.org/docs/latest/programming-guide.html#transformations), [map](http://spark.apache.org/docs/latest/programming-guide.html#transformations) y [reduceByKey](http://spark.apache.org/docs/latest/programming-guide.html#transformations) para computar las cuentas por palabra
en el archivo como un RDD de pares (String, Int). Para colectar las palabras en nuestra consola, podemos usar la acción
[collect](http://spark.apache.org/docs/latest/programming-guide.html#actions):

```
scala> wordCounts.collect()
res6: Array[(String, Int)] = Array((means,1), (under,2), (this,3), (Because,1), (Python,2), (agree,1), (cluster.,1), ...)
```

#### En Python:

```
>>> textFile.map(lambda line: len(line.split())).reduce(lambda a, b: a if (a > b) else b)
15
```

Esto primero mapea una línea a un valor entero, creado un nuevo RDD. `reduce` es llamado en ese RDD para encontrar la mayor cuenta en una línea. Los argumentos `map` y `reduce` son [funciones anónimas(lambdas)](https://docs.python.org/2/reference/expressions.html#lambda) de Python, pero también podemos pasar cualquier
función de alto nivel de Python que queramos. Por ejemplo, definiremos una función `max` para hacer más fácil de entender este código:

```
>>> def max(a, b):
...     if a > b:
...         return a
...     else:
...         return b
...

>>> textFile.map(lambda line: len(line.split())).reduce(max)
15
```

Un patrón común para flujo de datos es MapReduce, popularizado
por Hadoop. Spark puede implementar flujos de MapReduce fácilmente:

```
>>> wordCounts = textFile.flatMap(lambda line: line.split()).map(lambda word: (word, 1)).reduceByKey(lambda a, b: a+b)
```

Aquí, hemos combinado las transformaciones [flatMap](http://spark.apache.org/docs/latest/programming-guide.html#transformations), [map](http://spark.apache.org/docs/latest/programming-guide.html#transformations) y [reduceByKey](http://spark.apache.org/docs/latest/programming-guide.html#transformations) para computar las cuentas por palabra
en el archivo como un RDD de pares (string, int). Para colectar las palabras en nuestra consola, podemos usar la acción
[collect](http://spark.apache.org/docs/latest/programming-guide.html#actions):

```
>>> wordCounts.collect()
[(u'and', 9), (u'A', 1), (u'webpage', 1), (u'README', 1), (u'Note', 1), (u'"local"', 1), (u'variable', 1), ...]
```

## Almacenamiento en Caché [Caching]
