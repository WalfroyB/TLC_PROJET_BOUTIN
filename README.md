# README  
# Techniques logicielles pour le Cloud Computing - Projet  

Pour plus de précisions sur le travail effectué, se reporter au fichier `CR_PROJET.md` situé dans le dépôt Docker.
  
## 1 - Déploiement réalisé  
 
### 1.1 - Schéma et diagramme   

![](https://codimd.math.cnrs.fr/uploads/upload_bee6e77e1e57b3f1b6b26bb09acb23fc.png)  
  
Diagramme :  
  
![](https://codimd.math.cnrs.fr/uploads/upload_37488022f9564abab2f8942759f6452c.png)  
  
### 1.2 - Liens des trois applications, et dépôts utilisés     
  
Doodle https://walfroy.carryboo.io.  
Etherpad https://etherpad-walfroy.carryboo.io  
Phpmyadmin https://myadmin-walfroy.carryboo.io (db/tlc/tlc).  
  
Dépôt GitHub: https://github.com/WalfroyB/TLC_PROJET_BOUTIN  
Dépôts Docker Hub:  
    - https://hub.docker.com/repository/docker/walfvonfroy/image_front/general  
    -  https://hub.docker.com/repository/docker/walfvonfroy/image_back/general
  
## 2 - Fichiers dockerfile  
  
  Les tailles de mes deux images (front et back) sont les suivantes:  
  ![](https://codimd.math.cnrs.fr/uploads/upload_a742b020986fba2812112db4cff8585e.png)  
  
## 2.1 - Dockerfile du frontend (Angular)  
  
Ce dockerfile utilise le multi-stage en m'inspirant de ce [lien](https://stackoverflow.com/questions/64842509/deploying-angular-app-in-container-fails-to-run-ng-command).
  
```dockerfile
# dockerfile for image_front:v1.0 

# Stage 1

 # J'utilise node:10-alpine comme image de base pour mon contneur, pour cette première partie qui est le build  
FROM node:10-alpine AS build-step

 # Dans le conteneur, je créé un dossier app pour le front de mon application, qui devient le répertoire courant
WORKDIR /app

 # copie du fichier package.json du répertoire local (sur la machine hôte) dans le répertoire /app dans le conteneur Docker (cf note 06)
COPY package.json /app

 # Je lance npm install et non npm ci (cf note 07) pour installer les dépendances spécifiées dans le package.json, dans le dossier /app du conteneur
RUN npm install

 # copie de tout le contenu du répertoire local (sur la machine hôte) dans le répertoire /app du conteneur Docker
COPY . /app

 # Lancement du build
 # Les deux premiers -- indiquent à Docker que tous les arguments qui suivent sont des arguments pour la commande build. Les deux derniers -- définissent une option pour build: stockage de tous les fichiers résultants du build dans le répertoire /app/dist/out du conteneur Docker (cf note 08)
RUN npm run build -- --outputPath=./dist/out



# Stage 2

# J'utilise l'image de base nginx:1-alpine pour construire mon image docker
FROM nginx:1-alpine AS prod

 # De l'étape 1, je copie tous ce qu'il y a dans /app/dist/out/ (chemin absolu!) dans /usr/share/nginx/html.
 # Le répertoire de base de Nginx (/usr/share/nginx/html) est l'emplacement par défaut où les fichiers HTML, CSS, JavaScript et autres ressources statiques sont servis par Nginx. Cela signifie que si nous plaçons les fichiers générés dans ce répertoire, Nginx sera en mesure de les servir en réponse aux requêtes HTTP entrantes, sans avoir besoin de configuration supplémentaire
COPY --from=build-step /app/dist/out/ /usr/share/nginx/html
```
A NOTER: je rajoute ``"skipLibCheck"= true``, au fichier `tsconfig.json`, sinon j'ai une erreur lors du build.  
  
  ## 2.1 - Dockerfile du backend (Quarkus)  
  
Ce dockerfile utilise également le multi-stage, pour produire lors du BUILD, puis ne conserver que le `tlcdemoApp-1.0.0-SNAPSHOT-runner.jar`  
``` dockerfile
# dockerfile for image_back:v7.0 

# Stage 1 - BUILD
FROM alpine:3.17 AS build-step
RUN apk add --no-cache git maven && \
    git clone https://github.com/WalfroyB/TLC_PROJET_BOUTIN.git && \
    rm -rf TLC_PROJECT_BOUTIN/front
WORKDIR /TLC_PROJET_BOUTIN/api
RUN mvn clean package -Dquarkus.package.type=uber-jar

# Stage 2 - RUN
FROM alpine:3.17 AS prod
RUN apk add --no-cache openjdk11
WORKDIR /app
COPY --from=build-step /TLC_PROJET_BOUTIN/api/target/tlcdemoApp-1.0.0-SNAPSHOT-runner.jar /app/
CMD ["java", "-jar", "tlcdemoApp-1.0.0-SNAPSHOT-runner.jar"]
```
## 2.2 - Docker-compose
  
  Le docker-compose permettra, depuis la machine distante, d'installer et de démarrer les applications conteneurisées dans `container_front`, `container_api`, `container_mail`, `container_db`, `container_etherpad` et `container_phpmyadmin`.    
  
Point particulier:  afin que le conteneur `db` (qui met du temps à ce lancer) soit prêt au moment où le conteneur `api` démarre, j'ai rajouté dans la description de la `db` un `healthcheck`, qui permet à `api` de démarrer uniquement à la condition que le conteneur `db` soit `healthy`.   
  
```YAML
version: "3.8"

services:

  front:
    image: walfvonfroy/image_front:7.0 # image DockerHub
    container_name: container_front
    hostname: front
    ports:
      - "15080:80" # port standard pour le protocole HTTP
    depends_on:
      - phpmyadmin
      - etherpad
      - api
    restart: always
    volumes:
      - ./back.conf:/etc/nginx/conf.d/back.conf

  api:
    image: walfvonfroy/image_back:7.0
    container_name: container_api
    hostname: api
    depends_on:
      db:
        condition: service_healthy
    restart: on-failure

  db:
    image: mysql
    container_name: container_db
    hostname: db
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=tlc
      - MYSQL_USER=tlc
      - MYSQL_PASSWORD=tlc
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "--user=$$MYSQL_USER", "--password=$$MYSQL_PASSWORD"]
      interval: 4s
      timeout: 4s
      retries: 15
    volumes:
      - ./migration:/docker-entrypoint-initdb.d

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: container_phpmyadmin
    hostname: phpmyadmin
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_USER=tlc
      - MYSQL_PASSWORD=tlc
      - PMA_ARBITRARY=1
      - PMA_PMADB=phpmyadmin
      - PMA_HOST=db
    restart: always
    depends_on:
      - db

  etherpad:
    image: etherpad/etherpad:latest
    container_name: container_etherpad
    hostname: etherpad
    volumes:
      - ./APIKEY.txt:/opt/etherpad-lite/APIKEY.txt

  mail:
    image: bytemark/smtp
    container_name: container_mail
    hostname: mail
    restart: always
    depends_on:
      - api

volumes:
  db-data:
 ```
# FIN 
