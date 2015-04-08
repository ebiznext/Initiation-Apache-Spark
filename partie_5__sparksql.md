# Partie 5 : SparkSQL

Cette partie a pour but de vous faire prendre en main les APIs Spark SQL qui, à la différence de Spark Core, permet de gérer des flux de données **structurées** (avec un Schéma explicite ou définissable)

## Théorie
L'abstraction principale de Spark, les RDDs, n'ont aucune notion de schéma, on peut gérer des données sans aucune information de typage ou de noms de champs.

Pour permettre de gérer une notion de schéma (une liste de noms de champs avec leur type associé) Spark SQL a introduit dans les versions < 1.3, la notion de ```SchemaRDD``` qui est un ```RDD``` standard mais avec cette notion de schéma et donc des méthodes additionnelles.

Avec la nouvelle release de Spark 1.3.0, ce concept a été généralisé et renommé sous le nom de `DataFrame`.


## Exercice 8 : Appliquer un schéma à un RDD existant par *reflection*
Quand vous avez déjà un RDD avec des données structurées - une classe dédié comme ci-après la classe ```Movie```, vous pouvez appliquer un schéma et transformer le flux en **DataFrame**

Dans cet exercice nous allons parser un flux non-structuré via un RDD standard, le transformer en un ```RDD<Movie>``` pour enfin lui appliquer un schéma par reflection !

Tout d'abord voilà la classe du modèle :

```java
package org.devoxx.spark.lab.devoxx2015.model;

public class Movie implements Serializable {

    public Long movieId;
    public String name;

    public Product(Long movieId, String name) {
        this.movieId = movieId;
        this.name = name;
    }
}
```

Puis le code de travail Spark :

```java
public class Workshop8_Dataframe {

    public static void main(String[] args) throws URISyntaxException {
        JavaSparkContext sc = new JavaSparkContext("local", "Dataframes");

        String path = Paths.get(Workshop8_Dataframe.class.getResource("/products.txt").toURI()).toAbsolutePath().toString();

        JavaRDD<Movie> moviesRdd = sc.textFile(path)
                .map(line -> line.split("\\t"))
                .map(fragments -> new Movie(Long.parseLong(fragments[0]), fragments[1]));

        SQLContext sqlContext = new SQLContext(sc);

        // reflection FTW !
        DataFrame df = // TODO utiliser Movie.class pour appliquer un schema 

        df.printSchema();
        df.show(); // show statistics on the DataFrame
    }
}
```

En Scala dans **Workshop8.scala** : 

```scala
case class Movie(movieId: Long, name: String)

val moviesRDD = sc.textFile("products.txt").map(line => {
    val fragments = line.split("\\t")
    Movie(fragments(0).toLong, fragments(1))
})

val df = ...;// TODO utiliser Movie.class pour appliquer un schema 
        
df.show(); // show statistics on the DataFrame
```

## Exercice 9 : Charger un fichier Json

```java
public class SchemaRddHandsOn {

    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext();
        SQLContext sqlContext = new org.apache.spark.sql.SQLContext(sc);
        
        DataFrame df = ...// Charger le fichier products.json avec l'API sc.load(...)
        
        df.show(); // show statistics on the DataFrame
    }
}
```

En Scala dans **Workshop9.scala** :
```scala
object Workshop9 {

    def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setAppName("Workshop").setMaster("local[*]")
        val url = Paths.get(getClass.getResource("/ratings.txt").toURI).toAbsolutePath.toString
        val sc = new SparkContext(conf)
    
        val sqlContext: SQLContext = new org.apache.spark.sql.SQLContext(sc)
        import sqlContext.implicits._
        
        val df = ...// Charger un fichier json avec l'API sqlContext.load(...) --> DataFrame
        df.show() // show statistics on the DataFrame
    }
```

## Quelques opérations sur une DataFrame
La nouvelle API des ```DataFrame``` a été calquée sur l'API de [Pandas](http://pandas.pydata.org/) en Python. Sauf que bien sûr a la différence de Pandas, les ```DataFrame``` sont parallélisées.

```java 
df.printSchema(); // affiche le schéma en tant qu'arbre
df.select("name").show(); // n'affiche que la colonne name
df.select("name", df.col("id")).show(); // n'affiche que le nom et l'id
df.filter(df("id") > 900).show();
df.groupBy("product_id").count().show();

// chargement :
sqlContext.load("people.parquet"); // parquet par default sauf si overrider par spark.sql.sources.default
sqlContext.load("people.json", "json");
```

## Exercice 10 : SQL et tempTable(s)
Dans cet exercice le but va être d'utiliser notre schéma ainsi définit dans le contexte Spark SQL pour executer via Spark des requêtes SQL

La première requête va être de : 
** Trouver tout les films dont le nom contient Hook ** 

```java
public class SchemaRddHandsOn {

    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext();
        SQLContext sqlContext = new org.apache.spark.sql.SQLContext(sc);
        
        DataFrame df = sqlContext.load("products.json", "json");
        
        // On met à disposition en tant que table la DataFrame
        df.registerTempTable("movies");
        
        // ceci est une transformation !
        DataFrame hooks = sqlContext.sql(...); // TODO ajouter la requête SQL
        
        hooks.show();
    }
}
```

```scala
object Workshop10 {

  // Combien de fois chaque utilisateur a voté
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Workshop").setMaster("local[*]")
    val url = Paths.get(getClass.getResource("/ratings.txt").toURI).toAbsolutePath.toString
    val sc = new SparkContext(conf)

    val sqlContext: SQLContext = new org.apache.spark.sql.SQLContext(sc)
    import sqlContext.implicits._

    val df = sqlContext.load("products.json", "json");
    df.registerTempTable("movies");
    // ceci est une transformation !
    DataFrame hooks = sqlContext.sql(...); // TODO ajouter la requête SQL
    
    hooks.show()
  }
}
```

## Exercice 11 : de RDD vers DataFrame

```scala
object Workshop11 {

  // Combien de fois chaque utilisateur a voté
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Workshop").setMaster("local[*]")
    val url = Paths.get(getClass.getResource("/ratings.txt").toURI).toAbsolutePath.toString
    val sc = new SparkContext(conf)

    val sqlContext: SQLContext = new org.apache.spark.sql.SQLContext(sc)
    import sqlContext.implicits._

    val lines: RDD[Rating] = // ...
    
    val df = // Convertir le RDD en DataFrame
    val maxRatings: DataFrame = // renvoyer les utilisateurs ayant noté 5
    val sqlCount = // Combien y a t-il eu de ratings ?
    println(sqlCount.head().getLong(0))

    // TOP 10 utilisateurs par nombre de votes et les imprimer
    val sqlTop10 = //
  }
}
```

