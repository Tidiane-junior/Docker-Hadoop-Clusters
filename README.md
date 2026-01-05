# Hadoop - Docker
**Hadoop** est système distribué c-à-d un ensemble de machines(noeuds) gérées par **Namenode**. Il distribue les données aux **Datanodes** (noeuds).
Hadoop est composé de deux parties : 
    - **HDFS**(système de **gestion de fichiers** distribués): Il sert au stockages des big data. c'est un ensemble de fichiers et non une Base de Données.
    - **MapReduce** : le framework de Hadoop. Ilest à la fois un framework **d'execution parallelle et de traitement** de Données.

## Installation de Docker et des noeuds

Pour installer le logiciel Docker, merci de suivre les consignes disponibles ici https://docs.docker.com/desktop/, 
en fonction de votre système d’exploitation.

A partir de maintenant et pour la suite des TPs, il vous faudra penser à lancer Docker (qui s’exécutera en arrière plan).

Nous allons utiliser, tout au long de ce TP, **trois contenaires** représentant respectivement : 
**un nœud maître (le Namenode)** et **deux nœuds esclaves (les Datanodes)**.

## 1- Téléchargez l’image docker
Depuis un Terminal, tape : 

    docker pull liliasfaxi/docker-cluster-hadoop-spark-python-8:3.6

Si non utilisez celle-ci: 

    docker pull liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6

Ce container contient une distribution **Linux/Ubuntu, et les librairies nécessaires pour utiliser Hadoop et Spark**. 
Il contient également **python3.x** (version du langage python compatible avec les versions de Hadoop et Spark installées).

## 2- Créez les 3 contenaires à partir de l’image téléchargée. 
Pour cela:
### a. Créez un réseau qui permettra de relier les trois contenaires:

	docker network create --driver=bridge hadoop
	
### b. Créez et lancez les trois contenaires 

Les instructions **-****p** permettent de faire un mapping entre les ports de la machine hôte et ceux du contenaire.
Important Dans la suite, adaptez la syntaxe -16 à la syntaxe -8, en fonction de l’image que vous avez téléchargée.

1er contenaire : 	 
--
    docker run -itd --net=hadoop -p 9870:9870 -p 8088:8088 -p 7077:7077 -p 16010:16010 -p 9999:9999\
     --name hadoop-master --hostname hadoop-master liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6
2em contenaire : 	 
--
	docker run -itd -p 8040:8042 --net=hadoop --name hadoop-slave1\
     --hostname hadoop-slave1 liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6*
3em contenaire : 	 
--

    docker run -itd -p 8041:8042 --net=hadoop --name hadoop-slave2\
     --hostname hadoop-slave2 liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6

## Remarques

Sur certaines machines, la première ligne de commande ne s’exécute pas correctement. L’erreur provient sans doute du **port 9870** que doit déjà être utilisé par une autre application installée sur votre machine. 
Vous pouvez alors supprimer ce port de la première ligne de commande :

    docker run -itd --net=hadoop -p 8088:8088 -p 7077:7077 -p 16010:16010 -p 9999:9999 --name hadoop-master\ 
    --hostname hadoop-master liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6
		
Le **port 9999** sera utilisé lors du TP sur la librairie Spark streaming.

## Preparation au TP
 Entrez dans le contenaire hadoop-master pour commencer à l’utiliser : 
 --
    docker exec -it hadoop-master bash

 Le résultat de cette exécution sera le suivant: 
 --
    root@hadoop-master:~#

 Vous pouvez alors entrer dans le Namenode : 
 
    docker exec -it hadoop-master bash
 
 puis : 
 
    root@hadoop-master:~# ./start-Hadoop.sh
 
Il s’agit du *shell* ou du *bash* (Linux/Ubuntu) du **nœud maître**. 
   - La commande **ls**, qui liste les fichiers et dossiers du dossier en cours, doit faire état des répertoires et fichiers suivants (et d’autres fichiers éventuellement): **TP_Hadoop TP_Spark hdfs**

## Remarque 2

 Ces étapes de configuration ne doivent être réalisées qu’une seule fois. 
 Pour relancer le cluster (une fois qu’on a fermé puis relancé son ordinateur par ex.), il suffira :
  - de lancer l’application *Docker Desktop*, qui lance les daemon Docker.
  - de lancer la commande suivante : 
  
   		 docker start hadoop-master hadoop-slave1 hadoop-slave2

# Vid3 : Manipulation de fichiers dans HDFS

 Une fois Hadoop lancé, on affiche les fichiers avec la commande : 
 --

    ls -l 

Pour pouvoir manipuler les fichiers, on va utiliser la commande : 
--

    hadoop fs

Il va permettre d'uliliser **hdfs** comme si on était directement sur une seule machine. 
Créons une dossier imput apr exemple avec **-p** qui va prendre le path jusqu'à arriver à ce chemin.
Sans ce **-p** hadoop ne va rien créer car rien n'existe encore.
--

    hadoop fs -mkdir -p input

Il faut au préalable exécuter la commande : **./start-hadoop.sh** qui va lancer tous les **daemons** necessaires.
On peut vérifier le dossier **input** avec la commande : 

    hadoop fs -ls

On va exécuter le fichier **yarn** qui va gérer les ressources (allouer les ressouces) des job dans hadoop.
Nous allons copier le fichier **purchase.txt** que nous avons dans notre machine. Pour le voir on tape :

    ls

Pour le copier dans **input**, on fait : 
--

    hdfs dfs -put purchases.txt input

Pour l'afficher, on tape :
--

    hdfs dfs -ls input

Pour voir la structure du fichier **input/purchases.txt**, on tape :
--

    hdfs dfs -tail input/purchases.txt

--
## Interfaces web pour Hadoop

Hadoop offre plusieurs interfaces web pour pouvoir observer le comportement de ses différentes composantes. Il est possible d'afficher ces pages directement sur notre machine hôte, et ce grâce à l'utilisation de l'option *-**p* de la commande *docker run*. En effet, cette option permet de publier un port du contenaire sur la machine hôte.

En regardant la commande docker run utilisée plus haut, vous verrez que deux ports de la machine maître ont été exposés:

    - Le port 9870: qui permet d'afficher les informations de votre namenode.
    - Le port 8088: qui permet d'afficher les informations du resource manager de Yarn et visualiser le comportement des différents jobs.

Une fois votre cluster lancé et hadoop démarré et prêt à l'emploi, vous pouvez, sur votre navigateur préféré de votre machine hôte, aller à : http://localhost:9870.

## Vid4 : MapReduce avec Java

### Présentation

Un **Job Map-Reduce** se compose principalement de deux types de programmes:

 - **Mappers** : permettent d’extraire les données nécessaires sous forme de clef/valeur, pour pouvoir ensuite les trier selon la clef;

 - **Reducers** : prennent un ensemble de données triées selon leur clef, et effectuent le traitement nécessaire sur ces données (somme, moyenne, total...)

### Wordcount

Nous allons tester un programme **MapReduce** grâce à un exemple très simple, le *WordCount*, l'équivalent du *HelloWorld* pour les applications de traitement de données. Le Wordcount permet de calculer le nombre de mots dans un fichier donné, en décomposant le calcul en deux étapes:

    - L'étape de Mapping, qui permet de découper le texte en mots et de délivrer en sortie un flux textuel, où chaque ligne contient le mot trouvé, suivi de la valeur 1 (pour dire que le mot a été trouvé une fois)
    - L'étape de Reducing, qui permet de faire la somme des 1 pour chaque mot, pour trouver le nombre total d'occurrences de ce mot dans le texte.