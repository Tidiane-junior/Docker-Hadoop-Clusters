# TP Hadoop sur Docker – De 0 à HDFS + MapReduce
## Objectifs du TP

 À la fin tu sauras :
  - Lancer un cluster Hadoop avec Docker
  - Envoyer un fichier dans HDFS
  - Lire les données depuis HDFS
  - Lancer un job MapReduce
  - Récupérer les résultats
  
## Prérequis
 On doit avoir :
  - ✅ Docker Desktop installé
  - ✅ Docker Compose actif
  - ✅ Un terminal (PowerShell, Bash, etc
  
# 🚀 1) Installer Hadoop en pseudo-distribué avec Docker Desktop

Le plus smooth, c’est d’utiliser un container Hadoop préconfiguré.
On va partir sur l’image bigtop (propre, stable, classique en cours).

##👉 a) Crée un docker-compose.yml

Dans un dossier de ton choix :
--

	version: "3"
	services:
	  hadoop:
		image: apache/hadoop:3
		container_name: hadoop-pseudo
		hostname: hadoop-pseudo
		ports:
		  - "9870:9870"   # NameNode UI
		  - "9864:9864"   # DataNode UI
		  - "8088:8088"   # ResourceManager UI
		tty: true
		
## 👉 b) Lance le container
--

	docker-compose up -d
    
Ce code est à lancer une fois.

## ⚙️ 2) Démarrer les services Hadoop (namenode, datanode, etc.)
--

	docker exec -it hadoop-pseudo bash

## 1. Vérifie vraiment l’état du conteneur
--

	docker ps -a

### Formate le NameNode (1ère fois uniquement) : 
--

	hdfs namenode -format
	
### Démarre les daemons HDFS :
--

	start-dfs.sh
	
### Démarre les YARN HDFS :
	**start-yarn.sh**

### 🧪 3) Vérifier avec jps :
--
	jps*

## Ajouter un fichier local dans Namenode
--

	docker cp note.txt namenode:/tmp/

## Vérifier l'Ajout
--

	docker exec -it namenode ls -l /tmp

## Lire le contenu du fichier avec cat
--

	docker exec -it namenode cat /tmp/note.txt

## Vérifier que ton dossier HDFS existe
--

	docker exec -it namenode hdfs dfs -ls /

## Envoyer un fichier dans HDFS avec put
--

	docker exec -it namenode hdfs dfs -put /tmp/note.txt /donnees/

## Pour supprimer un dossier -r 
--

	docker exec -it namenode hdfs dfs -rm -r /donnees/test

## Récupérer le fichier du conteneur vers le local
### Étape A — Extraire le fichier HDFS dans le conteneur namenode
--

	docker exec -it namenode hdfs dfs -get /chemin_fichier_HDFS /chemin_fichier_namenode

### Étape B — Copier le fichier du conteneur vers PC
--
    
    docker cp namenode:/tmp/notes.txt .**
 
## Remplacer un fichier existant par un nouveau
Hadoop accepte l’option **-f** avec put pour remplacer automatiquement.
--

    docker exec -it namenode hdfs dfs -put -f /tmp/notes.txt /donnees/