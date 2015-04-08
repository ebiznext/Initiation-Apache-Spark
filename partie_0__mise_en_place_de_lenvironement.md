# Partie 0 : Mise en place de l'environement

## Pré-requis 
Pour bien commencer, il vous faut installer sur la machine :  
 
 * un JDK >= 1.8.x 
 * Maven >= 3.x
 * SBT
 * un IDE ou éditeur de texte : IntelliJ IDEA, Sublime Text, Eclipse... 
 * Fichiers de build : http://www.ebiznext.com/devoxx2015/build.zip

##Vérifier l'installation
```bash
$ cd spark
$bin/spark-shell

Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 1.4.0-SNAPSHOT
      /_/
         

scala> sc.parallelize(1 to 1000).foreach(println)
```

## Industrialisation en Java

Nous allons créer un nouveau projet maven avec Spark Core:

```shell
mkdir hands-on-spark-java/
cd hands-on-spark-java/
```

Dans le `pom.xml` : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.devoxx.spark.lab</groupId>
    <artifactId>devoxx2015</artifactId>
    <version>1.0-SNAPSHOT</version>

    <repositories>
        <repository>
            <id>Apache Spark temp - Release Candidate repo</id>
            <url>https://repository.apache.org/content/repositories/orgapachespark-1080/</url>
        </repository>
    </repositories>
    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.10</artifactId>
            <version>1.3.1</version>
            <!--<scope>provided</scope>--><!-- cette partie là a été omise dans notre projet pour pouvoir lancer depuis maven notre projet -->
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- we want JDK 1.8 source and binary compatiblility -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Industrialisation en Scala 

Pour créer un projet Scala, il vous faut :

```shell
mkdir hands-on-spark-scala/
cd hands-on-spark-scala/
```

```scala
name := "Devoxx2015"

version := "1.0"

scalaVersion := "2.11.2"

crossScalaVersions := Seq(scalaVersion.value)


resolvers += "Local Maven Repository" at "file://" + Path.userHome.absolutePath + "/.m2/repository"

resolvers += "sonatype-oss" at "http://oss.sonatype.org/content/repositories/snapshots"

resolvers += "Typesafe Releases" at "http://repo.typesafe.com/typesafe/releases/"

resolvers += "Conjars" at "http://conjars.org/repo"

resolvers += "Clojars" at "http://clojars.org/repo"


val sparkV = "1.3.0"

libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % sparkV,
)

Seq(Revolver.settings: _*)

 test in assembly := {}
  
 assemblyOption in assembly := (assemblyOption in assembly).value.copy(includeScala = false)
```

Avec dans un fichier `project/assembly.sbt` contenant : 
```scala
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.12.0")
```

Pour utiliser le plugin **SBT-Assembly** qui permet de générer via la commande `sbt assembly` un jar livrable pour Spark ne contenant ni Spark, ni Scala, mais toutes les autres dépendances nécessaires au projet.

## Récupération de Spark en tant que projet
Pour pouvoir soumettre le projet à un cluster en local, il vous faut télécharger ou récupérer par clé USB le binaire pré-compilé de Spark.

Si vous ne l'avez pas récupérer par une clé autour de vous, vous pouvez [télécharger Spark 1.3.0 en cliquant ici !](http://www.apache.org/dyn/closer.cgi/spark/spark-1.3.0/spark-1.3.0-bin-hadoop2.4.tgz) 

Pour installer à proprement dit Spark, il vous suffit de dézipper le fichier télécharger.
