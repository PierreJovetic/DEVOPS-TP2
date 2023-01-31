# TP1 DEVOPS : DOCKER

On commence par créer un dossier par services (et donc conteneur) nécessaire au fonctionnement de l'application :

* app_db : pour la base de données
* app_api : pour l'api
* app_http : Pour le serveur web

Pour voir la liste des dockers qui sont en train de tourner :
`sudo docker ps`   
Pour supprimer complètement un docker :
`sudo docker rm -f <NOM_DOCKER>`   
Pour accéder à un docker déjà lancé :
`sudo docker exec -it adminer/ID_CONTENEUR /bin/sh`    
Pour voir les logs d'un docker :
`sudo docker logs (-f) <NOM_DOCKER>`      
Pour voir les statistiques d'un dokcer :
`sudo docker stats <NOM_DOCKER>`    
Pour voir les docker qui appartiennent à un réseau :
`sudo docker inspect network <NOM_NETWORK>`


## Base de données

Tout d'abord on va créer le network dans lequel va se trouver nos deux dockers (postgres et adminer) :
```
docker network create app-network
```
### Initialisation de la base de données
On crée un Dockerfile pour la création du docker de la base de données PostgreSQL :
```shell
FROM postgres:14.1-alpine 

ENV POSTGRES_DB=db \
	POSTGRES_USER=usr \
	POSTGRES_PASSWORD=pwd

COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d
COPY 02-InsertData.sql /docker-entrypoint-initdb.d
```
Pour que les tables et les données soient automatiquement intégrée à la bdd, on copie les scripts SQL dans le dossier ```/docker-entrypoint-initdb.d```. Ce dossier est automatiquement exécuter par le docker Postgres à son inititialisation.

On build ensuite le Dockerfile pour avoir une image custom :
```
docker build -t pierre/db .
```
On lance ensuite le docker, en indiquant le réseau dans lequel il se trouve :
```
docker run -d --net=app-network --name app-db pierre/db
```
### Visualisation de la base de données
On créer aussi un docker ```adminer``` pour pouvoir visualiser ce qu'il y a dans la bdd :
```shell
# On pull l'image
docker pull adminer
```
```shell
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer
# On indique encore une fois le réseau dans lequel il sera pour qu'il puisse communiquer avec le docker que l'on a crée précédemment
```
On pourra donc visualiser la bdd depuis l'adresse `http://localhost:8090`  

### Persistance des données


Voici un exemple appliqué à un serveur d'argument pour assurer la persistence de ses données.
Par défaut si un chemin relatif `/var/lib/docker/volumes/`.

```sh
-v app-data:/var/lib/postgresql/data -d marc.llop/database 
# - v chemin/volume/local:/chemin/conteneur
```

 sudo docker run --name app-database -p 5432:5432 --net app-network -e POSTGRES_PASSWORD="toto" 

## Backend API


### Changement du mot de passe par défaut pour plus de "sécurité"
sudo docker exec -it app-database /bin/sh

psql -d db -U usr -h 127.0.0.1 -w toto -p 5432

## Api


```shell
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build # 
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```


## Serveur Web
