# Techniques logicielles pour le Cloud Computing - Projet  

**Rendu projet** : 24/03/2023 - 23h59 sur  https://forms.gle/ypoaNwh9FmwC2kyk7  
**Enoncé du projet** : https://hackmd.diverse-team.fr/s/SJqu5DjSD#projet  
  
[Vidéo 1](https://drive.google.com/file/d/1GQbdgq2CHcddTlcoHqM5Zc8Dw5o_eeLg/preview): application type Doodle (sondage, agenda, notepad partagé, synchro calendrier, ..), en QUARKUS côté serveur (BackEnd) et en ANGULAR côté client (FrontEnd).  
  
**But projet :**  
- conteneuriser chaque microservice de l'application ;  
- automatiser leur déploiement.  
  
[Vidéo 2](https://drive.google.com/file/d/1l5UAsU5_q-oshwEW6edZ4UvQjN3-tzwi/preview): IMPORTANT pour la compréhension, présentation de l'architecture de l'application et explications des attendus du projet de l'étape 1 à l'étape 5:  
  
- Etape 1: connection sécurisée entre mon navigateur et serveur web (Nginx) avec Letsenscrypt, MEP firewall, configurer un fail to ban, déployer sur un même serveur les services mail, BdD, pad ;  
- Etape 2: conteneuriser les 3 services pour faciliter leur déploiement ;  
- Etape 3: utilisation d'un orchestrateur (Microk8s) ;  
- Etape 4: MEP du monitoring avec Promotheus et Grafana (dashboard) ;  
- Etape 5: load balacing pour un ensemble d'instances de backend.  
- IMPORTANT: intégration continue, CI/CD automatisée  
  
[Vidéo 3](https://drive.google.com/file/d/1jxYNfJdtd4r_pDbOthra360ei8Z17tX_/preview): revue de code de l'application (```/api``` pour le back et ```/front```).  
  Dans le dossier ```api/sources/Docker```, fourniture d'embryon de dockerfile à compléter.  
  Code de l'application disponible à https://github.com/barais/doodlestudent.  
  
# Tâche 0:
**Enoncé:** Créer une machine virtuelle (https://vm.istic.univ-rennes1.fr/ séléctionner ubuntu20 comme image de base) et demandez l’accès externe vers le port 80 (http) et 443 (https) de votre machine virtuelle par le helpdesk, catégorie ISTIC-ESIR - Tous problèmes informatiques, l’accès au port 22 se fera au travers du VPN. Partager moi l’adresse IP de la machine et le sous domaine souhaité ici (sous domaine de diverse-team.fr). (Je mets jour quand c’est fait le google doc joint)
  
Je prévois deux machines sur https://vm.istic.univ-rennes1.fr/ :  
- walf09.istic.univ-rennes1.fr (148.60.11.60) ;  
- walf10.istic.univ-rennes1.fr (148.60.11.139).  

Demande d'accès externe vers le port 80 (http) et 443 (https), TICKETS n°278436 et n°278434 :  
  
![](https://codimd.math.cnrs.fr/uploads/upload_3fe8e54374486bca7ffcee5101a87aa9.png)  
  
![](https://codimd.math.cnrs.fr/uploads/upload_f6f992dfe58ce4bc35e8191d4c8b21ea.png)  

Noms de domaines demandés: 
- http://boutin.diverse-team.fr associé à walf09 (148.60.11.60) ;  
- http://boutintest.diverse-team.fr associé à walf10 (148.60.11.139).  

**Installation de Docker sur mon serveur**
 - j'active FortiClient ;  
 - Je démarre ma VM ISTIC ;  
 - je me connecte en SSH sur ma VM :  
```ssh zprojet@walf10.istic.univ-rennes1.fr```  
    >`sudo apt-get update`  
    >`sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y`  
    >`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`  
    >`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`  
        TAPER "ENTREE"
    >`sudo apt-get update`  
    >`sudo apt-get install docker-ce docker-ce-cli containerd.io -y`  
    >`sudo usermod -aG docker zprojet`  
    >`sudo docker run hello-world`
    INSTALLATION DE DOCKER-COMPOSE
    >`sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`  
    >`sudo chmod +x /usr/local/bin/docker-compose`  
    >`docker-compose --version`  
    CONFIGURATION D'UN RESEAU DOCKER nommé `network-tlc-projet`  
    >`sudo docker network create network-tlc-projet`  
    Je vérifie que les ports 80 et 443 ne sont pas utilisés:  
    >`sudo lsof -i :80`  
    >`sudo lsof -i :443`  
    Si besoin je tue les processus avec leur PID  
    >`sudo kill $PID`  
    
**Installation de Nginx sur mon serveur**

**ou Installation d'un conteneur Nginx sur mon serveur**  
  
  - Installation
    >`docker run --name nginx-container --network network-tlc-projet -p 80:80 -p 443:443 -d nginx`  
- Vérification de bon fonctionnement :
    >`sudo docker ps -a`  
    >`sudo lsof -i :80`  
    >`sudo lsof -i :443`  
    ![](https://codimd.math.cnrs.fr/uploads/upload_74dfd9cb19025c772fc033c9f8aa7fd6.png)
    
    Dans mon navigateur, je vérifie que le serveur fonctionne sur : http://148.60.11.139 (le port 80 étant le port par défaut, ça fonctionne).    
    ![](https://codimd.math.cnrs.fr/uploads/upload_16c1385e8b8843a70232e44fab26f1f0.png)
 
OK JUSQUE LA.  
Pour désinstaller Nginx:  
```
sudo systemctl stop nginx
sudo systemctl disable nginx
sudo apt-get remove nginx
sudo apt-get purge nginx
sudo rm -rf /etc/nginx
sudo apt autoremove
```
# Tâche 1  

Je "fork" le projet sur mon GitHub: https://github.com/WalfroyB/TLC_PROJET_BOUTIN.  
  ![](https://codimd.math.cnrs.fr/uploads/upload_4cf2c9ee61e324eb88c738ac0c596c4e.png)  
  
**Enoncé** :   
   - Fournir un ou plusieurs docker file(s) et un ou plusieurs docker compose(s) pour cette application permettant de facilement déployer et configurer l’application.  
  - Vous aurez du mal à avoir un fonctionnement correct à cette étape là. En effet, le code du front va faire ces requêtes REST à la même adresse que celui qui lui a fourni le code html, css et js pour éviter les problèmes de CORS. Il est donc nécessaire de se forcer à configurer le serveur nginx qui délivre le front pour faire proxy_pass quand il reçoit une requête sur la route /api ou une sous route de /api. Ne vous inquiétez pas, on configure cela à l’étape suivante.  
  
## 1.1-Partie client ou FRONTEND  
  
  Le dossier ```/front``` est le frontend de mon application. Il est développé en ANGULAR.  
  
  Je consulte https://angular.io/guide/setup-local .  

- Pré-requis :   
  Node.js. Je choisis une release 18 (LTS).  
  Gestionnaire de paquets node pm.  
    
### 1.1.1-Test d'installation manuelle  

Je teste l'installation manuelle sur mon serveur, puis en local (pour avoir l'interface graphique du front) avant l'utilisation des dockerfile et des docker compose.  
  
`sudo apt-get update`
`sudo apt-get install python3 g++ make python3-pip`
`sudo snap install curl`
`curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -` pour la NodeJS LTS latest - ne pas prendre sur la VM

`curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -` ne fonctionne pas en local
`curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash - &&\
sudo apt-get install -y nodejs` fonctionne ne local

`sudo apt-get install gcc g++ make`

`curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
     echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
     sudo apt-get update && sudo apt-get install yarn`
     
`sudo apt-get install -y nodejs`
![](https://codimd.math.cnrs.fr/uploads/upload_740e9ae2f7f30c21e348169158161acd.png)


```sudo npm install -g npm@9.6.0```  
```sudo npm install -g @angular/cli@10.1.7```  
![](https://codimd.math.cnrs.fr/uploads/upload_bf9ae56a1e9bc7d59d7c8a3807dd8c28.png)

`sudo apt-get install git -y`
```mkdir front```  
```git clone https://github.com/WalfroyB/TLC_PROJET_BOUTIN.git```  
```cp -r /home/zprojet/TLC_PROJET_BOUTIN/front/* /home/zprojet/front/```  
```sudo rm -rf TLC_PROJET_BOUTIN/```  
```cd ./front```   
// installation de toutes les dépendances du projet spécifiées dans le fichier package.json. 
```npm install```    
:::warning
```bash
npm WARN old lockfile 
npm WARN old lockfile The package-lock.json file was created with an old version of npm,
npm WARN old lockfile so supplemental metadata must be fetched from the registry.
npm WARN old lockfile 
npm WARN old lockfile This is a one-time fix-up, please be patient...
npm WARN old lockfile 
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated stable@0.1.8: Modern JS already guarantees Array#sort() is a stable sort, so this library is deprecated. See the compatibility table on MDN: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort#browser_compatibility
npm WARN deprecated source-map-resolve@0.5.3: See https://github.com/lydell/source-map-resolve#deprecated
npm WARN deprecated source-map-url@0.4.0: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated sourcemap-codec@1.4.8: Please use @jridgewell/sourcemap-codec instead
npm WARN deprecated streamroller@2.2.4: 2.x is no longer supported. Please upgrade to 3.x or higher.
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN deprecated readdir-scoped-modules@1.1.0: This functionality has been moved to @npmcli/fs
npm WARN deprecated read-package-tree@5.3.1: The functionality that this package provided is now in @npmcli/arborist
npm WARN deprecated querystring@0.2.0: The querystring API is considered Legacy. new code should use the URLSearchParams API instead.
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
npm WARN deprecated node-fetch-npm@2.0.4: This module is not used anymore, npm uses minipass-fetch for its fetch implementation now
npm WARN deprecated ini@1.3.5: Please update to ini >=1.3.6 to avoid a prototype pollution issue
npm WARN deprecated svgo@1.3.2: This SVGO version is no longer supported. Upgrade to v2.x.x.
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
npm WARN deprecated debug@4.2.0: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated date-format@3.0.0: 3.x is no longer supported. Please upgrade to 4.x or higher.
npm WARN deprecated protractor@7.0.0: We have news to share - Protractor is deprecated and will reach end-of-life by Summer 2023. To learn more and find out about other options please refer to this post on the Angular blog. Thank you for using and contributing to Protractor. https://goo.gle/state-of-e2e-in-angular
npm WARN deprecated @npmcli/move-file@1.0.1: This functionality has been moved to @npmcli/fs
npm WARN deprecated @schematics/update@0.1001.7: This was an internal-only Angular package up through Angular v11 which is no longer used or maintained. Upgrade Angular to v12+ to remove this dependency.
npm WARN deprecated tslint@6.1.3: TSLint has been deprecated in favor of ESLint. Please see https://github.com/palantir/tslint/issues/4534 for more information.
npm WARN deprecated chokidar@2.1.8: Chokidar 2 does not receive security updates since 2019. Upgrade to chokidar 3 with 15x fewer dependencies
npm WARN deprecated chokidar@2.1.8: Chokidar 2 does not receive security updates since 2019. Upgrade to chokidar 3 with 15x fewer dependencies
npm WARN deprecated date-format@2.1.0: 2.x is no longer supported. Please upgrade to 4.x or higher.
npm WARN deprecated debug@4.1.1: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@3.2.6: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@4.1.1: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@4.1.1: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@3.2.6: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@3.2.6: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@4.1.1: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated debug@4.1.1: Debug versions >=3.2.0 <3.2.7 || >=4 <4.3.1 have a low-severity ReDos regression when used in a Node.js environment. It is recommended you upgrade to 3.2.7 or 4.3.1. (https://github.com/visionmedia/debug/issues/797)
npm WARN deprecated core-js@3.6.4: core-js@<3.23.3 is no longer maintained and not recommended for usage due to the number of issues. Because of the V8 engine whims, feature detection in old core-js versions could cause a slowdown up to 100x even if nothing is polyfilled. Some versions have web compatibility issues. Please, upgrade your dependencies to the actual version of core-js.

added 1510 packages, and audited 1511 packages in 2m

71 packages are looking for funding
  run `npm fund` for details

76 vulnerabilities (3 low, 20 moderate, 39 high, 14 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.

```
:::
```ng build```  
 J'ai une erreur **ERR_OSSL_EVP_UNSUPPORTED**
 ![](https://codimd.math.cnrs.fr/uploads/upload_c730a05e6c9d2be1ecd9d233ff73e4ce.png)
`export NODE_OPTIONS=--openssl-legacy-provider`
Je vérifie que toutes les dépendances de mon projet sont à jour et je tape la commande :  
`npm outdated`
Puis je mets à jour chaque package non à jour:

@fullcalendar/angular                 5.3.1 to 5.11.4
`npm update @fullcalendar/angular`
Lors de cet update, j'ai l'indication suivante :  
![](https://codimd.math.cnrs.fr/uploads/upload_942b5793621028b6306ccfd1a577d7e0.png)
  Pourtant dans mon fichier ```package-lock.json```, j'ai bien :  
  ![](https://codimd.math.cnrs.fr/uploads/upload_010455ce6dace94313dfa24afb66f386.png)
`npm uninstall chokidar@2.1.8`
`npm install chokidar@3.4.3` 


@fullcalendar/interaction             5.3.1  to  5.11.4
`npm update @fullcalendar/interaction`

@fullcalendar/timegrid                5.3.1 to   5.11.4
`npm update @fullcalendar/timegrid`

@fullcalendar/daygrid                 5.3.2 to   5.11.4
`npm update @fullcalendar/daygrid` automatiquement mis à jour avec le précédent.

@types/jasminewd2                     2.0.8 to   2.0.10
`npm update @types/jasminewd2`

@types/node                        12.12.68  to  12.20.55
`npm update @types/node`

bootstrap                             4.5.3  to   4.6.2
`npm update bootstrap`

codelyzer                             6.0.1  to   6.0.2
`npm update codelyzer`

jquery                                3.5.1  to   3.6.3
`npm update jquery`

typescript                            4.0.3  to   4.0.8
`npm update typescript`

rxjs                                  6.6.3  to   6.6.7
`npm update rxjs`

primeicons                            4.0.0  to   4.1.0
`npm update primeicons`

karma-chrome-launcher                 3.1.0  to   3.1.1
`npm update karma-chrome-launcher`

karma-jasmine                         4.0.1  to    4.0.2
`npm update karma-jasmine`

karma-jasmine-html-reporter           1.5.4   to   1.7.0
`npm update karma-jasmine-html-reporter`

POUR CE DERNIER - PROBLEME avec jasmine-core
![](https://codimd.math.cnrs.fr/uploads/upload_bcdaa8d1e6fa014144b5ca7e91c7c38e.png)

`npm outdated | grep jasmine-core` donne:
```bash
Package       Current    Wanted   Latest  
jasmine-core  3.6.0      3.6.0    4.5.0  
```
Je teste avec jasmine-core@3.8
`npm install jasmine-core@3.8`
`npm install karma-jasmine-html-reporter`
`npm update`
`npm outdated` pour vérifier.

**Toutes les dépendances sont à jour.**

`sudo apt-get update`
`rm -rf node_modules`
`rm -rf package-lock.json` IMPORTANT A RETENIR!
`npm install`  
   
`ng build` NE FONCTIONNE TOUJOURS PAS.

`sudo apt-get update && sudo apt-get install openssl`

`ng build` NE FONCTIONNE TOUJOURS PAS.

Je tente:
`sudo apt-get upgrade -y`

Je tente l'installation de NodeJS v16.19.1 (LTS): 
`sudo apt-get remove nodejs`
`curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -`
`sudo apt-get install -y nodejs`

**Ca compile à peu près correctement.**

`ng build` indique deux erreurs : 
![](https://codimd.math.cnrs.fr/uploads/upload_c99637613cc0bba0960f779135723692.png).
`npm run e2e` indique également les deux même erreurs : 
![](https://codimd.math.cnrs.fr/uploads/upload_1d85a6022722d0dd7a0d45f5e6951333.png)  
```bash
ERROR in node_modules/preact/src/jsx.d.ts:2138:24 - error TS2304: Cannot find name 'SVGMPathElement'.

2138  	mpath: SVGAttributes<SVGMPathElement>;
      	                     ~~~~~~~~~~~~~~~
node_modules/preact/src/jsx.d.ts:2145:22 - error TS2304: Cannot find name 'SVGSetElement'.

2145  	set: SVGAttributes<SVGSetElement>;
      	                   ~~~~~~~~~~~~~
```  
Je rajoute `"skipLibCheck"= true,` au fichier `tsconfig.json`.

Je relance `ng build` et ça fonctionne.
![](https://codimd.math.cnrs.fr/uploads/upload_65866c7c5749cbce6c75d8d4b65d73db.png)


npm run e2e
![](https://codimd.math.cnrs.fr/uploads/upload_34d1e7b110d425ffa5d7e2d24b3ebd25.png)
![](https://codimd.math.cnrs.fr/uploads/upload_11bb6be02c7e92629671541ceaf9a77d.png)
![](https://codimd.math.cnrs.fr/uploads/upload_59a0de226233a30c201ed3dd8fb6faf0.png)

PB CHROME donc tentative de résolution:

`wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb`

`sudo apt-get install libwayland-server0 fonts-liberation libgbm1 libvulkan1` 

`sudo dpkg -i google-chrome-stable_current_amd64.deb`

`google-chrome --version`
**Google Chrome 110.0.5481.177**

Je vérife la version de Chrome Driver :
`chromedriver --version`

`wget https://chromedriver.storage.googleapis.com/LATEST_RELEASE`

`export CHROMEDRIVER_VERSION=$(curl -sS https://chromedriver.storage.googleapis.com/LATEST_RELEASE)`

`VERSION=$(cat LATEST_RELEASE)`

`wget https://chromedriver.storage.googleapis.com/$VERSION/chromedriver_linux64.zip`

`unzip chromedriver_linux64.zip`

`sudo mv chromedriver /usr/local/bin/`

`chromedriver --version`
ChromeDriver 110.0.5481.100
En relançant le test `npm run e2e`: j'ai le résultat suivant : 
```bash
E/launcher - unknown error: Chrome failed to start: exited abnormally.
  (unknown error: DevToolsActivePort file doesn't exist)  
  
E/launcher - WebDriverError: unknown error: Chrome failed to start: exited abnormally.
  (unknown error: DevToolsActivePort file doesn't exist)
  
E/launcher - Process exited with error code 199  
```
**Je mets de côté la résolution des problèmes du `npm run e2e`, le principal est que j'arrive à builder !**

Je lance `ng test` et le résultat est automatique malgré quelques erreurs: Chrome se lance.   
![](https://codimd.math.cnrs.fr/uploads/upload_420bea5e84f25f67c34f6998c624ab09.png)


### 1.1.2-Construction du dockerfile pour le front  
  
  Je consulte ce [lien](https://stackoverflow.com/questions/64842509/deploying-angular-app-in-container-fails-to-run-ng-command) pour un problème d'application ANGULAR (utilisant @angular/cli@10.1.7) dans un CONTAINER. Le dockerfile proposé en réponse, dont je vais me servir comme base, est le suivant :   
```dockerfile
FROM node:10-alpine as build-step

WORKDIR /app
COPY package.json .
RUN npm ci
COPY . .
RUN npm run build --prod

# Stage 2
FROM nginx:1-alpine
COPY --from=build-step /app/ /usr/share/nginx/html
```
  
Je consulte la page https://docs.docker.com/build/building/multi-stage/, non vue en TP DOCKER, car c'est ce que propose le dockerfile ci-dessus.  

Je me place dans mon dossier `front` :  
  - mon dockerfile (en deux parties - multi-staging) est le suivant :  
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
  
A NOTER: je rajoute ``"skipLibCheck"= true``, au fichier tsconfig.json, sinon j'ai une erreur lors du build.  
- je construis mon image docker `image_front`, taggée v1.0 :   
```sudo docker build -t image_front:image_front_v1.0 -f dockerfile .```  
- je l'enregistre dans image_front.tar  
`sudo docker save image_front:v1.0 -o image_front.tar`  
`sudo chmod a+rwx image_front.tar`  
- je lance mon conteneur:  
`sudo docker run -p 80:80 -p 443:443 image_front:image_front_v1.0`  

- je teste le bon fonctionnement en local :  
  http://localhost:80
![](https://codimd.math.cnrs.fr/uploads/upload_5d4112ff7c493a1984c51707e4d1bf2c.png)  

**Pour démarrer le conteneur automatiquement, j'utiliserais plus tard un docker-compose.**  

## 1.2-Partie BACKEND  
  
La partie Back-end est développée en Quarkus.  
  
### 1.2.1 - Test en local

Je vais consulter le fichier `pom.xml` et j'analyse: 
```xml
<artifactId>tlcdemoApp</artifactId>
	<version>1.0.0-SNAPSHOT</version>
```
Celà veut dire qu'il va produire un fichier `tlcdemoApp-1.0.0-SNAPSHOT.jar` et non un fichier `code-with-quarkus-1.0.0-SNAPSHOT-runner.jar` comme indiquée dans le [Readme.md](https://github.com/barais/doodlestudent/blob/main/api/README.md).  
  
Je lance, depuis `/api` :  
- le packagin de l'application
`./mvnw package`  
Malgré quelques WARNING, j'obtiens:  
![](https://codimd.math.cnrs.fr/uploads/upload_f7f8c8bcd55e18f08c0eb7abf34fafaf.png)  
  
- l'exécution de l'application  
`java -jar target/tlcdemoApp-1.0.0-SNAPSHOT.jar`  
![](https://codimd.math.cnrs.fr/uploads/upload_3b6c3c86a9d07fcc8f4a3bb79ef0bf8f.png)  
- le fichier `tlcdemoApp-1.0.0-SNAPSHOT.jar` n'est pas exécutable, car il faut modifier son fichier `manifest.mf`. 
  
- **Je change de méthode et installe GraalVM**:
Je télécharge dans mes programmes `graalvm-ce-java19-linux-amd64-22.3.1.tar.gz`  
Je l'extrais: `tar xvzf graalvm-ce-java19-linux-amd64-22.3.1.tar.gz`  
Je vais dans mon dossier `/api`:  
```bash
export JAVA_HOME=/home/walfroy/Programmes/graalvm-ce-java19-22.3.1
export PATH=$JAVA_HOME/bin:$PATH
gu install native-image
```
- Je modifie le fichier `pom.xml` et rajoute une ligne `<additionalBuildArgs>-J-Xmx2G</additionalBuildArgs>` pour spécifier avec l'argument `-J-Xmx2G` une taille de mémoire maximale de 2 Go pour la machine virtuelle Java dans la construction de Maven (sinon ça me donne des erreurs), dans:   
```xml
    </systemPropertyVariables>
    <additionalBuildArgs>-J-Xmx2G</additionalBuildArgs>
</configuration>
```    
- Je créé un exécutable natif et je le lance:   
`sudo ./mvnw package -Pnative`  
ou
`sudo ./mvnw package -Pnative -Dquarkus.native.native-image-xmx=2g`
puis
`./target/tlcdemoApp-1.0.0-SNAPSHOT-runner
`  
  
  ![](https://codimd.math.cnrs.fr/uploads/upload_4404e69f970b03a7853492c9769a4696.png)
  
Lorsque je me rends sur http://localhost:8080, j'obtiens : 
  
![](https://codimd.math.cnrs.fr/uploads/upload_74c08413f35130088fca4f6e7d21fa04.png)
  
  
### 1.2.2 - Lancement du backend via un dockerfile  

Je consulte https://quarkus.io/guides/building-native-image.  
   
N'arrivant pas à construire un dockerfile qui aille au bout du build, je teste en local la troisième méthode:  
```
./mvnw clean package -Dquarkus.package.type=uber-jar
```
```
java -jar target/tlcdemoApp-1.0.0-SNAPSHOT-runner.jar
```
```
sudo docker build -t image_back:v1.0 -f dockerfile .
```
```
sudo docker run -p 80:80 -p 443:443 image_back:v1.0
```
Celà fonctionne encore en local.  
  
Je construis le dockerfile suivant:  
```dockerfile
# dockerfile for image_back:v1.0 

# Utilisation d'une image de base alpine 
FROM alpine:3.17

# Installation de trois paquets, git, maven et openjdk11
RUN apk add git openjdk11 maven --no-cache

# Clonage du repo GitHub source et suppression de la partie front
RUN git clone https://github.com/WalfroyB/TLC_PROJET_BOUTIN.git
RUN rm -rf TLC_PROJECT_BOUTIN/front

# Définition du répertoire de travail pour les commandes suivantes
WORKDIR TLC_PROJET_BOUTIN/api

# Construction d'un über-jar pour mon application
RUN mvn clean package -Dquarkus.package.type=uber-jar

# Exposition du port 8080
EXPOSE 8080

# Lancement de l'application
CMD [ "java", "-jar", "target/tlcdemoApp-1.0.0-SNAPSHOT-runner.jar" ]
```

Je le simplifie pour avoir le moins de commandes possibles:  
```dockerfile
# dockerfile for image_back:v1.0 

FROM alpine:3.17
RUN apk add --no-cache git openjdk11 maven && \
    git clone https://github.com/WalfroyB/TLC_PROJET_BOUTIN.git && \
    rm -rf TLC_PROJECT_BOUTIN/front
WORKDIR /TLC_PROJET_BOUTIN/api
RUN mvn clean package -Dquarkus.package.type=uber-jar
EXPOSE 8080
CMD [ "java", "-jar", "target/tlcdemoApp-1.0.0-SNAPSHOT-runner.jar" ]
```
Je lance les commandes:  
`sudo docker build -t image_back:image_back_v1.0 -f dockerfile .`
`sudo docker run -p 80:80 -p 443:443 image_back:image_back_v1.0`
  
Sur http://localhost:8080, j'obtiens ma page front. 
  
J'optimise mon dockerfile en taggant une v2.0 en faisant du multi-stagging:  
```dockerfile
# dockerfile for image_back:v2.0 

# Stage 1 - BUILD
FROM alpine:3.17 AS build-step
RUN apk add --no-cache git maven && \
    git clone https://github.com/WalfroyB/TLC_PROJET_BOUTIN.git && \
    rm -rf TLC_PROJECT_BOUTIN/front
WORKDIR /TLC_PROJET_BOUTIN/api
RUN mvn clean package -Dquarkus.package.type=uber-jar

# Stage 2 - RUN
FROM alpine:3.17 AS prod
RUN apk add --no-cache openjdk11 && \
WORKDIR /app
COPY --from=build-step /TLC_PROJET_BOUTIN/api/target/tlcdemoApp-1.0.0-SNAPSHOT-runner.jar /app/
EXPOSE 8080
CMD ["java", "-jar", "tlcdemoApp-1.0.0-SNAPSHOT-runner.jar"]
```  
  
La différence entre les deux images `image_back:image_back_v1.0` et `image_back:image_back_v2.0` est de 264MB:  
![](https://codimd.math.cnrs.fr/uploads/upload_7cbd1f95fc7777a4e61c4d77c07220a8.png)
  
  
## 1.3-Partie docker-compose    
    
Tout d'abord je vais placer mes images docker dans deux répertoires de mon Docker Hub :  
![](https://codimd.math.cnrs.fr/uploads/upload_31675e3a72914962c234bf1d2c53bd79.png)
  
Mes images sont les suivantes:  
![](https://codimd.math.cnrs.fr/uploads/upload_4aeb8a1ecb9735ed011aafb2f2c52d77.png)  
  
Pour la partie back  
Etape 1 - TAG  
```
# Après tag, <nom_image>:<tag> <nom_répertoire>
sudo docker tag image_back:image_back_v2.0 walfvonfroy/tlc_image_back_v2.0
```
Etape 2 - PUSH  
```
sudo docker push walfvonfroy/tlc_image_back_v2.0
```
Pour la partie front  
```
sudo docker tag image_front walfvonfroy/tlc_image_front_v1.0
sudo docker push walfvonfroy/tlc_image_front_v1.0
``` 
![](https://codimd.math.cnrs.fr/uploads/upload_4521c6ed56330e71a05593f51a90171e.png)

Mes images disponibles sur Docker Hub seront disponibles avec les noms `walfvonfroy/tlc_image_front_v1.0` et `walfvonfroy/tlc_image_back_v2.0`.  

Les conteneurs seront `image_front:image_front_v1.0` et `image_back:image_back_v2.0`.  

Lorsque je lance mon conteneur du front en local avec `sudo docker run -p 80:80 walfvonfroy/tlc_image_front_v1.0
`, je constate qu'un conteneur docker est lancé et que mon front est disponible à http://localhost:80 .  
![](https://codimd.math.cnrs.fr/uploads/upload_7f75fb965737970aed4936abe943b471.png)


Lorsque je lance mon conteneur du back en local avec `sudo docker run -p 8080:8080 walfvonfroy/tlc_image_back_v2.0`je constate qu'un conteneur docker est lancé et que mon back est disponible à http://localhost:8080 .  
![](https://codimd.math.cnrs.fr/uploads/upload_3f7b30f082ade827c688a72f959570ba.png)

Maintenant, je vais construire mon `docker-compose.yml` à l'aide de l'exemple donné.
  
https://docs.docker.com/compose/compose-file/compose-file-v3/#depends_on  
Je consulte sur `depends_on` et il est mentionné que le service `back`, avec `depends_on` pour la `db`, n'attend pas que `db` soit “prête” avant de lancer le `back`, et que je peux contrôler l'ordre de démarrage avec `healthcheck`.  
  
https://docs.docker.com/compose/compose-file/#restart  
Ce `healthcheck` doit être effectué jusqu'à ce que ça fonctionne, avec `restart: on-failure` ou `restart: always`.  
  
https://docs.docker.com/compose/compose-file/#healthcheck  
Donc je vais d'abord m'assurer que le service `db` soit démarré et sain avec un `healthcheck`, pour que le service `back` puisse être lancé correctement à son tour.  
  
Pour la commande de `test`, je trouve des explications sur https://stackoverflow.com/questions/54479941/how-to-init-mysql-database-in-docker-compose/54480687#54480687  
  
Pour ne pas perdre les données, j'inclue au service `db` un volume `db-data` monté sur `/var/lib/mysql` qui stockera les données de la base de données.  
  
Je consulte également les liens pour :   
- db (image:mysql): https://hub.docker.com/_/mysql
- mail (image:bytemark/smtp) https://hub.docker.com/r/bytemark/smtp
- etherpad (image:etherpad/etherpad:latest) https://hub.docker.com/r/etherpad/etherpad
- phpmy admin (image:phpmyadmin/phpmyadmin:latest) https://hub.docker.com/layers/phpmyadmin/phpmyadmin/latest/images/sha256-0d951ee3bec76c5d7083122f5db509ebfa6c209efc5e70f1d47af2e13a34f543?context=explore

```YAML
version: "3.8"
services:
  
  front:
    image: walfvonfroy/tlc_image_front_v1.0:latest # image DockerHub
    container_name: container_front
    hostname: front
    restart: always
    ports:
      - "80:80" #port standard pour le protocole HTTP
      - "443:443" #port standard pour le protocole HTTPS
      
  backend:
    image: walfvonfroy/tlc_image_back_v2.0:latest
    container_name: container_back
    hostname: api
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy
    restart: on-failure
      
  db:
    image: mysql
    container_name: conainer_db
    hostname: db
    ports:
      - "3306:3306"
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

  etherpad:
    image: etherpad/etherpad:latest
    container_name: container_etherpad
    hostname: etherpad
    ports:
      - "9001:9001"
    volumes:
      - ./APIKEY.txt:/opt/etherpad-lite/APIKEY.txt

  mail:
    image: bytemark/smtp
    container_name: container_mail
    hostname: mail
    restart: always
    ports:
      - "2525:25"
      
volumes:
  db-data:

```

Lorsque je lance la comande `sudo docker-compose up -d` dans mon dossier `/api`, j'obtiens:  
![](https://codimd.math.cnrs.fr/uploads/upload_f96ae4d5cde1f40e72486894a227f60b.png)  
  
Je teste les ports en localhost:  
![](https://codimd.math.cnrs.fr/uploads/upload_cda71c3ff3044b43f6603b49f8a67db1.png)

Maintenant, je dois résoudre le problème des ports:  
`0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 8080/tcp`  
`0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp`  
`0.0.0.0:9001->9001/tcp, :::9001->9001/tcp`  
`0.0.0.0:2525->25/tcp`  
  
`0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 8080/tcp` signifie que:  
- Le port 8000 du container est lié (ou "mapped") au port 8000 de l'hôte (0.0.0.0:8000->8000/tcp) : cela permet d'accéder à l'API du backend en utilisant l'adresse IP de l'hôte et le port 8000.  
- Le port 8000 du container est également lié à une adresse IPv6 (:::8000->8000/tcp).  
- Le port 8080/tcp est également exposé par le container, mais n'est pas lié à un port de l'hôte. Cela signifie que si l'application à l'intérieur du container écoute sur le port 8080, il est possible de s'y connecter depuis d'autres containers dans le même réseau Docker.  

Etant donné que pour mon back, je vais utiliser une VM (walf09) dont l'adresse IP est 148.60.11.60 (http://walf09.istic.univ-rennes1.fr ), je dois changer le 0.0.0.0.   

Dans api/src/main/resources/application.yml, qui est le fichier de configuration, je vais modifier `internalPadUrl: "http://localhost:9001/"` en  `internalPadUrl: "http://148.60.11.60:9001/"`.

**Utile** `sudo docker rm -f container_back container_front container_phpmyadmin container_db container_etherpad container_mail`
  
# - Tâche 2

Je vais me servir d'un fichier nginx du front pour router les requêtes, ainsi il sera plus facile à modifier.  
  
## 2.1 - Ecriture des fichier de conf
  
Dans`/api`, je créé un dossier `conf` dans lequel je vais créer trois fichiers:  
- `etherpad.conf`:  du nom de mon service `etherpad` dans le `docker-compose.yml`:
- `back.conf`:  du nom de mon service `back` dans le `docker-compose.yml`:
- `phpmyadmin.conf`:  du nom de mon service `phpmyadmin` dans le `docker-compose.yml`:
Je vais utiliser les lignes de code données pour la configuration des trois services dans un fichier `nginx.conf`.  
En première approche, je vais devoir indiquer:
- les noms de domaine ; 
- les conteneurs associés ;  
- autres modifications 1 ;
- autresmodifications 2 .

```nginx.conf
server {
    listen       80;
    listen  [::]:80;
    # server name to change based on your own domain name for doodle
    #server_name  doodle.tlc.fr;
    server_name  boutin.diverse-team.fr;
    
    location / {
        proxy_pass http://front:80;
        proxy_set_header Host $http_host;
        try_files $uri $uri/ /index.html?$args;
        
    location /api {
        proxy_pass http://back:8080/api;
        proxy_set_header Host $http_host;
    }
    
    
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html?$args;

    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server {
    listen       80;
    listen  [::]:80;
    # server name to change based on your own domain name for doodle
    server_name  myadmin.tlc.fr;

    location / {
        proxy_pass http://myadmin:80;
        proxy_set_header Host $http_host;

    }
 
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server {
    listen       80;
    listen  [::]:80;
    # server name to change based on your own domain name for doodle
    server_name  pad.tlc.fr;

    location / {
        proxy_pass http://etherpad:9001;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
	proxy_set_header X-Real-IP  $remote_addr;
	proxy_set_header X-Forwarded-For $remote_addr;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
  
  
## 2.2 - Modification du dockerfile du front  
  
J'ai créé un fichier `nginx.conf` (cf 2.1) que je vais placer dans le même répertoire que le `dockerfile` du front.  
  
  Je modifie le `dockerfile` du front (à la dernière ligne) pour permettre la prise en compte de ce fichier `nginx.conf`, de la manière suivante :  
```dockerfile
# dockerfile for image_front:v1.0 

# Stage 1 - BUILD

FROM node:10-alpine AS build-step
WORKDIR /app
COPY package.json /app
RUN npm install
COPY . /app
RUN npm run build -- --outputPath=./dist/out

# Stage 2 - PROD

FROM nginx:1-alpine AS prod
COPY --from=build-step /app/dist/out/ /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
```    

Je push cette version 2.0 de mon dockerfile sur mon Docker Hub:  
``` dockerfile
sudo docker build -t walfvonfroy/image_front:2.0 -f dockerfile .
sudo docker push walfvonfroy/image_front:2.0
``` 

## 2. - Mise en ligne de `nginx.conf` et du dockerfile du front  
  
Je push mon fichier `nginx.conf` sur mon GitHub.
    
  Je push ma version 2.0 de mon dockerfile sur mon Docker Hub:  
``` dockerfile
sudo docker build -t image_front:image_front_v2.0 -f dockerfile .
sudo docker run -p 80:80 image_front:image_front_v2.0
sudo docker tag image_front walfvonfroy/tlc_image_front_v2.0
sudo docker push walfvonfroy/tlc_image_front_v2.0
``` 
  
# - Tâche 3

https://walfroy.carryboo.io
https://etherpad-walfroy.carryboo.io
https://myadmin-walfroy.carryboo.io



# - Tâche 4
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
**Notes personnelles**:  
  
**Note 01**: Quarkus est un framework d'application Java, conçu pour fonctionner nativement avec Kubernetes, OpenJDK HotSpot et GraalVM. Il offre une faible empreinte mémoire et un temps de démarrage réduit.  
  
  **Note 02**: Un framework est un ensemble d'outils, de bibliothèques et de conventions qui permettent aux développeurs de créer plus rapidement et plus facilement des applications. Il fournit une structure et des fonctionnalités de base qui permettent aux développeurs de se concentrer sur les aspects spécifiques de l'application qu'ils sont en train de développer.  
  
Les frameworks offrent une abstraction de haut niveau pour les développeurs, qui peuvent ainsi se concentrer sur le code métier de leur application plutôt que sur les détails de la mise en œuvre des fonctionnalités de bas niveau. Les frameworks sont souvent basés sur des modèles de conception éprouvés et offrent des fonctionnalités prêtes à l'emploi pour la gestion des requêtes, la gestion des bases de données, la gestion des sessions utilisateur, la sécurité, etc.  
  
Dans le cas de Quarkus, il s'agit d'un framework conçu pour faciliter la création d'applications Java cloud-native, c'est-à-dire des applications qui peuvent s'exécuter efficacement dans des environnements de conteneurisation tels que Kubernetes. Quarkus fournit des fonctionnalités spécifiques pour la mise en œuvre de microservices, telles que la prise en charge des conteneurs, la gestion des requêtes HTTP, la gestion des événements, la mise en cache, etc.  
  
**Note 03**:  Angular est un framework pour clients, open source, qui permet la création d’applications Web et plus particulièrement d'applications Web monopages : des applications Web accessibles via une page Web unique qui permet de fluidifier l’expérience utilisateur et d’éviter les chargements de pages à chaque nouvelle action.
   
**Note 04**:  Pour voir si Node.js et npm sont déjà installés et vérifier la version installée:  
  
**Note 05**:   Pour vérifier l'utilisation d'un port sur m machine locale (exemple le port 80), rentrer la commande ```sudo lsof -i :80```.
    
**Note 06**:  différence entre `package.json` et `package-lock.json`   
   
`package.json` et `package-lock.json` sont deux fichiers utilisés par l'outil `Node.js Package Manager (npm)` pour gérer les dépendances de votre projet Node.js.  
  
Le fichier `package.json` contient une liste de toutes les dépendances de votre projet, ainsi que d'autres informations telles que le nom, la version et la description de votre application. Ce fichier peut être mis à jour manuellement ou via des commandes npm telles que `npm install` ou `npm uninstall`. Lorsque vous exécutez la commande `npm install`, npm lit le fichier `package.json` pour déterminer les dépendances nécessaires à installer. Cependant, le fichier `package.json` ne spécifie pas de manière exhaustive les versions des dépendances à installer, et cela peut causer des problèmes de compatibilité entre les versions des dépendances.  
  
C'est là que `package-lock.json` intervient. Ce fichier est généré automatiquement par `npm` lorsqu'il installe les dépendances de votre projet. Il contient une liste exhaustive de toutes les dépendances de votre projet, ainsi que les versions précises installées et les dépendances de niveau inférieur. Ce fichier garantit que les mêmes versions de dépendances sont installées sur tous les environnements et assure ainsi la reproductibilité des builds. Le fichier `package-lock.json` est généralement vérifié dans votre système de contrôle de version pour garantir que tous les développeurs utilisent la même configuration de dépendances.  
  
En résumé, `package.json` est utilisé pour spécifier les dépendances de votre projet et leurs versions compatibles, tandis que `package-lock.json` est utilisé pour garantir que les mêmes versions de dépendances sont installées sur tous les environnements de build.  
  
**Note 07**:  différence entre `npm ci` et `npm install`  

La commande `npm ci` est une commande de l'outil `Node.js Package Manager (npm)` qui installe les dépendances de votre projet à partir du fichier `package-lock.json`. Contrairement à la commande `npm install`, la commande `npm ci` est conçue pour les environnements de production, où il est important de garantir que les dépendances sont installées de manière cohérente et sans ambiguïté.

La commande `npm ci` est plus rapide que npm install, car elle ignore le fichier `package.json` et ne résout pas les dépendances, mais installe directement les versions spécifiées dans le fichier `package-lock.json`. Cela garantit que les mêmes versions de dépendances sont installées à chaque fois, sans tenir compte des mises à jour potentielles de versions mineures. En conséquence, cette commande est utile pour garantir la reproductibilité des builds et pour éviter les problèmes liés aux mises à jour non intentionnelles des dépendances.  
  
Il est important de noter que `npm ci` supprime tout d'abord le répertoire `node_modules`, puis installe les dépendances spécifiées dans `package-lock.json`. Donc si j'ajoute de nouvelles dépendances à mon projet, je dois m'assurer de mettre à jour `package-lock.json` avant d'exécuter `npm ci`.  
  
**Note 08**:  
  
Le choix de placer les fichiers générés par le build dans `./dist/out` est une convention courante pour les applications web, et cela est souvent spécifié dans le fichier de configuration du build.  
  
Le répertoire `dist` est souvent utilisé comme destination de build pour les projets JavaScript, car il est considéré comme un répertoire de distribution, où se trouvent les fichiers optimisés pour une utilisation en production. Les fichiers dans ce répertoire sont souvent minifiés, concaténés et optimisés pour la performance.  
  
Le choix de créer un sous-dossier `out` est simplement une convention pour éviter de polluer le répertoire `dist` avec des fichiers de build temporaires ou d'autres fichiers non pertinents. Le sous-dossier `out` contient uniquement les fichiers nécessaires pour l'application web, qui sont prêts à être servis par le serveur web (dans ce cas, nginx).  
  
En résumé, le choix de placer les fichiers générés par le build dans `./dist/out` est une convention courante pour les applications web, qui permet de séparer les fichiers de build temporaires et les fichiers optimisés pour la production.    
  
**Note 09**:  
