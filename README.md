# Hadoop - Docker
**Hadoop** est système distribué c-à-d un ensemble de machines (noeuds) gérées par **Namenode**. Il distribue les données aux **Datanodes**(noeuds).
Hadoop est composé de deux parties : 
	**HDFS**(système de **gestion de fichiers** distribués): Il sert au stockages des big data. c'est un ensemble de fichiers et non une Base de Données.
	**MapReduce** : le framework de Hadoop. Ilest à la fois un framework **d'execution parallelle et de traitement** de Données.


#Installation de Docker et des noeuds

Pour installer le logiciel Docker, merci de suivre les consignes disponibles ici https://docs.docker.com/desktop/, 
en fonction de votre système d’exploitation.

A partir de maintenant et pour la suite des TPs, il vous faudra penser à lancer Docker (qui s’exécutera en arrière plan).

Nous allons utiliser, tout au long de ce TP, **trois contenaires** représentant respectivement : 
**un nœud maître (le Namenode)** et **deux nœuds esclaves (les Datanodes)**.

## 1- Téléchargez l’image docker, 
Depuis un Terminal, tape : **docker pull liliasfaxi/docker-cluster-hadoop-spark-python-8:3.6**
Si non utilisez celle-ci: **docker pull liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6**
Ce container contient une distribution **Linux/Ubuntu, et les librairies nécessaires pour utiliser Hadoop et Spark**. 
Il contient également **python3.x** (version du langage python compatible avec les versions de Hadoop et Spark installées).

## 2- Créez les 3 contenaires à partir de l’image téléchargée. 
Pour cela:
### a. Créez un réseau qui permettra de relier les trois contenaires:
	**docker network create --driver=bridge hadoop**
	
### b. Créez et lancez les trois contenaires 
Les instructions *-p* permettent de faire un mapping entre les ports de la machine hôte et ceux du contenaire.
Important Dans la suite, adaptez la syntaxe -16 à la syntaxe -8, en fonction de l’image que vous avez téléchargée.

**docker run -itd --net=hadoop -p 9870:9870 -p 8088:8088 -p 7077:7077 -p 16010:16010 -p 9999:9999\
 **--name hadoop-master --hostname hadoop-master liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6**

*docker run -itd -p 8040:8042 --net=hadoop --name hadoop-slave1\
 --hostname hadoop-slave1 liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6*

*docker run -itd -p 8041:8042 --net=hadoop --name hadoop-slave2 \
	**--hostname hadoop-slave2 liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6*

## Remarques

Sur certaines machines, la première ligne de commande ne s’exécute pas correctement. L’erreur provient sans doute du port 9870 
que doit déjà être utilisé par une autre application installée sur votre machine. 
Vous pouvez alors supprimer ce port de la première ligne de commande :
*docker run -itd --net=hadoop -p 8088:8088 -p 7077:7077 -p 16010:16010 -p 9999:9999 --name hadoop-master\
	**--hostname hadoop-master liliasfaxi/docker-cluster-hadoop-spark-python-16:3.6*
		
Le port 9999 sera utilisé lors du TP sur la librairie Spark streaming.

## Preparation au TP
 Entrez dans le contenaire hadoop-master pour commencer à l’utiliser : **docker exec -it hadoop-master bash**
 Le résultat de cette exécution sera le suivant: **root@hadoop-master:~#**
 Vous pouvez alors entrer dans le Namenode : **docker exec -it hadoop-master bash**
 puis : **root@hadoop-master:~# ./start-Hadoop.sh**
 
Il s’agit du *shell* ou du *bash* (Linux/Ubuntu) du **nœud maître**. 
- La commande ls, qui liste les fichiers et dossiers du dossier en cours, doit faire état des répertoires et fichiers suivants 
(et d’autres fichiers éventuellement):
	**TP_Hadoop TP_Spark hdfs**

## Remarque 2
 Ces étapes de configuration ne doivent être réalisées qu’une seule fois. 
 Pour relancer le cluster (une fois qu’on a fermé et relancé son ordinateur p. ex.), il suffira 
  - de lancer l’application *Docker Desktop*, qui lance les daemon Docker.
  - de lancer la commande suivante : **docker start hadoop-master hadoop-slave1 hadoop-slave2**

# Manipulation de fichiers dans HDFS
 
 