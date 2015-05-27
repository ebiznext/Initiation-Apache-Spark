# Partie 6 : Chargement et stockage des données

## Elasticsearch
ElasticSearch expose une interfe d'entrée sortie permettant de récupérer les données au format RDD et de sauvegarder les RDDs directement en JSON.

### Ajout d'Elasticsearch au contexte Spark
```Scala
   import org.apache.spark.{SparkConf, SparkContext}
    val conf = new SparkConf().setAppName(Settings.Spark.AppName).setMaster(Settings.Spark.Master)
    conf.set("es.nodes", "localhost")
    conf.set("es.port", "9200")
    val sc = new SparkContext(conf)
```

### Lecture des données dans Elasticsearch
```Scala
    val esWildcardQuery =  search in "myindex" -> "rating" query { matchall }

    // val ratings = sc.esRDD("myindex/rating")

    val allData: RDD[(String, Map[String, AnyRef])] = sc.esRDD("myindex/rating", "{'match_all' : { }}").persist()
```

### Sauvegarde des données dans Elasticsearch
```Scala
    allData.filter(_.rating < 5).saveToEs("myindex/rating")
```


## Cassandra - Connecteur
Cassandra possède maintenant son connecteur spécifique et avec l'introduction des `DataFrame` et du calcul de plan d'exécution qui vient avec a été codifié le **predictive pushdown**. TODO


```xml
<dependency>
    <groupId>com.datastax.spark</groupId>
    <artifactId>spark-cassandra-connector_2.10</artifactId>
    <version>1.2.0-rc3</version>
</dependency>
<dependency>
    <groupId>com.datastax.spark</groupId>
    <artifactId>spark-cassandra-connector-java_2.10</artifactId>
    <version>1.2.0-rc3</version>
</dependency>
```

ou en Scala en utilisant SBT : 
```scala
libraryDependencies ++= Seq(
"com.datastax.spark" %% "spark-cassandra-connector" % "1.2.0-rc3"
)
```

Ensuite pour charger une table Cassandra dans Spark :

```java
import static com.datastax.spark.connector.japi.CassandraJavaUtil.*;

public class CassandraTest{

    public static void main(String[] args) {
        JavaRDD<Double> pricesRDD = javaFunctions(sc).cassandraTable("ks", "tab", mapColumnTo(Double.class)).select("price");
    }
}
```

et en Scala

```scala
val conf = SparkConf(true)
    .set("spark.cassandra.connection.host", "127.0.0.1")
    
val sc = SparkContext("local[*]", "test", conf)

import com.datastax.spark.connector._

val rdd = sc.cassandraTable("test", "kv")

// ou encore :
val collection = sc.parallelize(Seq(("key3", 3), ("key4", 4)))

collection.saveToCassandra("test", "kv", SomeColumns("key", "value"))       

```

```scala
import org.elasticsearch.spark.sql._

val sql = new SQLContext(sc)
val rdd1 = ... // from ES
val rdd2 = ... // from ES
rdd1.registerTempTable("table1")
rdd2.registerTempTable("table2")
sqlContext.sql("SELECT username, COUNT(*) AS cnt FROM wikiData WHERE username <> '' GROUP BY username ORDER BY cnt DESC LIMIT 10").collect().foreach(println)


```


