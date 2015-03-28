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
Cassandra possède maintenant son connecteur spécifique et avec l'introduction des ```DataFrame``` et du calcul de plan d'exécution qui vient avec a été codifié le **predictive pushdown**. TODO


```xml
<dependency>
    <groupId>com.datastax.spark</groupId>
    <artifactId>spark-cassandra-connector_2.10</artifactId>
    <version>1.2.0-rc2</version>
</dependency>
<dependency>
    <groupId>com.datastax.spark</groupId>
    <artifactId>spark-cassandra-connector-java_2.10</artifactId>
    <version>1.2.0-rc2</version>
</dependency>
```
