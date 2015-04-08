# Partie 1 : Familiarisation avec les concepts de Spark

Cette partie a pour but de vous familiariser avec les concepts principaux de Spark. Tout les exercices de cette partie seront en [Java](http://java.org) et s'appuieront sur [Maven](http://maven.org), mais vous pouvez aussi lancer un [REPL Spark]() pour tester en live les concepts au fur et à mesure de la présentation.

## Historique
Pour comprendre Spark, il faut comprendre son contexte historique, Spark emerge près de 8 ans après les début du Map Reduce chez Google.

![Contexte historique de Spark](history.png)

## Concepts


### RDD
L'abstraction de base de Spark est le RDD pour Resilient Distributed Dataset, c'est une structure de donnée immutable qui représente un Graph Acyclique Direct des différentes opérations à appliquer aux données chargées par Spark.

Un calcul distribué avec Spark commence toujours par un chargement de données via un **Base RDD**.

Plusieurs méthodes de chargements existent, mais pour résumé, tout ce qui peut être chargé par Hadoop peut être chargé par Spark, on s'appuie pour se faire sur un `SparkContext` ici nommé `sc` :

* En premier lieu, on peut charger dans Spark des données déjà chargées dans une JVM via `sc.paralellize(...)`, sinon on utilisera
* `sc.textFile("hdfs://...")` pour charger des données fichiers depuis HDFS ou le système de fichier local
* `sc.hadoopFile(...)` pour charger des données Hadoop
* `sc.sequenceFile(...)` pour charger des SequenceFile Hadoop
* Ou enfin `sc.objectFile(...)` pour charger des données sérialisées.


### Transformations et Actions
2 concepts de base s'appuient et s'appliquent sur le RDD, 
+ les Transformations
+ les Actions


Les Transformations sont des actions **lazy** ou à évaluation paresseuse, elles ne vont lancer aucun calcul sur un Cluster.

De plus les RDD étant immutables, une transformations appliquée à un RDD ne va pas le modifier mais plutôt en **créer un nouveau enrichit de nouvelles informations correspondant à cette transformation**.

Une fois que vous avez définit toutes les transformations que vous voulez appliquer à votre donnée il suffira d'appliquer une Action pour lancer le calcul sur votre Cluster ou CPU locaux (selon le SparkContext utilisé *c.f. ci-après*)

![Transformations et Actions sur un RDD](transformations.png)

Le RDD ne correspond en fait qu'à une sorte de plan d'exécution contenant toutes les informations de quelles opérations vont s'appliquer sur quelle bout ou partition de données.

### Spark Context
Le SparkContext est la couche d'abstraction qui permet à Spark de savoir où il va s'exécuter. 

Un SparkContext standard sans paramètres correspond à l'exécution en local sur 1 CPU du code Spark qui va l'utiliser.

Voilà comment instantier un SparkContext en Scala : 
```scala
val sc = SparkContext()
// on peut ensuite l'utiliser par exemple pour charger des fichiers :

val paralleLines : RDD[String] = sc.textFile("hdfs://mon-directory/*")
```

En Java pour plus de compatibilité, et même si on pourrait utiliser un `SparkContext` standard, on va privilégier l'utilisation d'un `JavaSparkContext` plus adapté pour manipuler des types Java.

```java
public class SparkContextExample {
    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext();
        
        JavaRDD<String> lines = sc.textFile("/home/devoxx/logs/201504*/*.txt");
    }
}
```

## Exercice : créer son premier RDD
En java :
```java

public class FirstRDD {
    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext();
        
        JavaRDD<String> lines = sc.textFile("/home/devoxx/logs/201504*/*.txt");
        // TODO rajouter une action sans quoi aucun calcul n'est lancé
    }
}
```

En Scala ou dans le REPL Spark vous pouvez aussi faire : 
```scala
scala> sc.paralellize(1 to 1000).map( _ * 10)
// TODO rajouter une action sans quoi aucun calcul n'est lancé
```


## Exercice : soumettre un jar à spark-submit

