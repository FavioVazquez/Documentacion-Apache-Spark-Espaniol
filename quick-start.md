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

```python
>>> wordCounts.collect()
[(u'and', 9), (u'A', 1), (u'webpage', 1), (u'README', 1), (u'Note', 1), (u'"local"', 1), (u'variable', 1), ...]
```

## Almacenamiento en Caché [Caching]

Spark también soporta colocar conjuntos de datos en una caché
en memoría en todo el clúster. Esto es muy útil cuando los datos son accedidos repetidamente, como cuando hacemos búsquedas [queries] a un conjunto de datos pequeños "caliente" [hot] o cuando estamos corriendo un algoritmo iterativo tipo PageRank. Como un ejemplo simple, marquemos nuestro conjunto de datos `linesWithSpark` como almacenado en la caché [cached]:

#### En Scala:

```java
scala> linesWithSpark.cache()
res7: spark.RDD[String] = spark.FilteredRDD@17e51082

scala> linesWithSpark.count()
res8: Long = 19

scala> linesWithSpark.count()
res9: Long = 19
```

#### EN Python:

```python
>>> linesWithSpark.cache()

>>> linesWithSpark.count()
19

>>> linesWithSpark.count()
19
```

Puede parecer tonto utilizar Spark para explorar y poner en la caché a un achivo de texto de 100 líneas. La parte interesante es que estas mismas funciones pueden ser utilizadas en conjuntos de datos muy grandes, asún cuando estén distribuidos a lo largo de decenas o cententedas de nodos. Puedes hacer este también interactivamente conectando `bin/spark/shell` o `bin/pyspark` a un clúster, como se describe en la [guía de programación](http://spark.apache.org/docs/latest/programming-guide.html#initializing-spark).

## Aplicaciones Autosuficientes

Supongamos que queremos escribir una aplicación autosuficiente usando la API de Spark. Exploraremos una aplicación simple en Scala (con sbt), Java (con Maven), y Python.

#### En Scala:

Crearemos una aplicaión de Spark muy simple en Scala - muy simple, de hecho se llama SimpleApp.scala:

```
/* SimpleApp.scala */
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

object SimpleApp {
  def main(args: Array[String]) {
    val logFile = "YOUR_SPARK_HOME/README.md" // Should be some file on your system
    val conf = new SparkConf().setAppName("Simple Application")
    val sc = new SparkContext(conf)
    val logData = sc.textFile(logFile, 2).cache()
    val numAs = logData.filter(line => line.contains("a")).count()
    val numBs = logData.filter(line => line.contains("b")).count()
    println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
  }
}
```

Nota que las aplicaciones deben definir un método `main()` en cambio de extender `scala.App`. Las subclases de `scala.App` pueden no funcionar correctamente.

Este programa solo cuenta el número de líneas que contienen 'a' y el número de líneas que contienen 'b' en el README de Spark. Nota que necesitaras reemplazar YOUR_SPARK_HOME con el lugar donde Spark está instalado. No como los pasados ejemplos usando la consola de Spark, que inicializa su propio SparkContext, inicializaremos un SparkContext como una parte del programa.

Pasamos al constructor SparkContext un [SparkContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf) que contiene la información sobre nuestra aplicación.

Nuestra aplicación depende de la API de Spark, así que también incluimos un archivo de configuración sbt, `simple.sbt` que explica que Spark es una dependencia. Este archivo también agrega un repositorio del que Spark depende:

```java
name := "Simple Project"

version := "1.0"

scalaVersion := "2.10.4"

libraryDependencies += "org.apache.spark" %% "spark-core" % "1.5.0"
```

Para que sbt funcione correctamente, necesitaremos diseñar
`SimpleApp.scala` y `simple.sbt` de acuerdo con la estructura típica de directorios. Cuando ya está eso en lugar, podemos crear un paquete JAR que contiene el código de la aplicación, y luego usar el script `spark-submit` para correr nuestro programa.

```
# Your directory layout should look like this
$ find .
.
./simple.sbt
./src
./src/main
./src/main/scala
./src/main/scala/SimpleApp.scala

# Package a jar containing your application
$ sbt package
...
[info] Packaging {..}/{..}/target/scala-2.10/simple-project_2.10-1.0.jar

# Use spark-submit to run your application
$ YOUR_SPARK_HOME/bin/spark-submit \
  --class "SimpleApp" \
  --master local[4] \
  target/scala-2.10/simple-project_2.10-1.0.jar
...
Lines with a: 46, Lines with b: 23
```

#### En Java:

Este ejemplo usará Maven para compilar una aplicación JAR, pero cualquier sistema de compilado similar servirá. Crearemos una aplicación de Spark muy simple, SimpleApp.java:

```java
/* SimpleApp.java */
import org.apache.spark.api.java.*;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.function.Function;

public class SimpleApp {
  public static void main(String[] args) {
    String logFile = "YOUR_SPARK_HOME/README.md"; // Should be some file on your system
    SparkConf conf = new SparkConf().setAppName("Simple Application");
    JavaSparkContext sc = new JavaSparkContext(conf);
    JavaRDD<String> logData = sc.textFile(logFile).cache();

    long numAs = logData.filter(new Function<String, Boolean>() {
      public Boolean call(String s) { return s.contains("a"); }
    }).count();

    long numBs = logData.filter(new Function<String, Boolean>() {
      public Boolean call(String s) { return s.contains("b"); }
    }).count();

    System.out.println("Lines with a: " + numAs + ", lines with b: " + numBs);
  }
}
```
Este programa solo cuenta el número de líneas que contienen 'a' y el número de líneas que contienen 'b' en el README de Spark. Nota que necesitaras reemplazar YOUR_SPARK_HOME con el lugar donde Spark está instalado. Como en el ejemplo de Scala, inicializamos un SparkContext, pero usamos la clase especial `JavaSparkContext` para obtener uno acoplado a Java. También creamos RDDs (representados por `JavaRDD`) y corremos transformaciones en ellos. Finalmente, pasamos funciones a Spark creando clases y extendiendo `spark.api.java.function.Function`. La [guía de programación de Spark](http://spark.apache.org/docs/latest/programming-guide.html) describe estas diferencias en más detalle.

Para compilar el programa, también escribimos un `pom.xml` de Maven que lista Spark como una dependencia. Nota que los artefactos de Scala están etiquetados con la versión de Scala:

```xml
<project>
  <groupId>edu.berkeley</groupId>
  <artifactId>simple-project</artifactId>
  <modelVersion>4.0.0</modelVersion>
  <name>Simple Project</name>
  <packaging>jar</packaging>
  <version>1.0</version>
  <dependencies>
    <dependency> <!-- Spark dependency -->
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-core_2.10</artifactId>
      <version>1.5.1</version>
    </dependency>
  </dependencies>
</project>
```

Diseñamos estos archivos de acuerdo a la estructura canónica de directorios de Maven:

```
$ find .
./pom.xml
./src
./src/main
./src/main/java
./src/main/java/SimpleApp.java
```
Podemos ahora empaquetar la aplicación usando Maven y ejecutándola con `.bin/spark-submit`.

```
# Package a JAR containing your application
$ mvn package
...
[INFO] Building jar: {..}/{..}/target/simple-project-1.0.jar

# Use spark-submit to run your application
$ YOUR_SPARK_HOME/bin/spark-submit \
  --class "SimpleApp" \
  --master local[4] \
  target/simple-project-1.0.jar
...
Lines with a: 46, Lines with b: 23
```

#### En Python:

Ahora mostraremos como escribir una aplicación usando la API de Pyton (Pyspark). Como un ejemplo, crearemos una aplicación de Spark simple, SimpleApp.py:

```python
"""SimpleApp.py"""
from pyspark import SparkContext

logFile = "YOUR_SPARK_HOME/README.md"  # Should be some file on your system
sc = SparkContext("local", "Simple App")
logData = sc.textFile(logFile).cache()

numAs = logData.filter(lambda s: 'a' in s).count()
numBs = logData.filter(lambda s: 'b' in s).count()

print("Lines with a: %i, lines with b: %i" % (numAs, numBs))
```

Este programa solo cuenta el número de líneas que contienen 'a' y el número de líneas que contienen 'b' en el README de Spark. Nota que necesitaras reemplazar YOUR_SPARK_HOME con el lugar donde Spark está instalado. Como en los ejemplos de Scala y Javam usamos un SparkContext para crear RDDs. Podemos pasar funciones de Python a Spark, que son serializadas automáticamente junto con cualquier variable que ellas referencien. Para aplicaciones que usen clases propias o librerías externas, podemos también añadir dependencias de código en el `spark-submit` a través de si argumento `--py-files` empaquetándoos en un archivo .zip (ver `spark-submit --help` para más detalles). SimpleApp es lo suficientemente simple por lo que no necesitamos especificar ninguna dependencia de código.

Podemos correr esta aplicación usando el script `bin/spark-submit`:

```
# Use spark-submit to run your application
$ YOUR_SPARK_HOME/bin/spark-submit \
  --master local[4] \
  SimpleApp.py
...
Lines with a: 46, Lines with b: 23
```
