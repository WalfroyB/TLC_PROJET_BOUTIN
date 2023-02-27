# Techniques logicielles pour le Cloud Computing - Projet  

**Enoncé du projet** : https://hackmd.diverse-team.fr/s/SJqu5DjSD#projet  
  
[Vidéo 1](https://drive.google.com/file/d/1GQbdgq2CHcddTlcoHqM5Zc8Dw5o_eeLg/preview): application type Doodle (sondage, agenda, notepad partagé, synchro calendrier, ..), en QUARKUS côté serveur (BackEnd) et en ANGULAR côté client (FrontEnd).  
  
**But projet :**  
- conteneuriser chaque microservice de l'application ;  
- automatiser leur déploiement.  
  
[Vidéo 2](https://drive.google.com/file/d/1l5UAsU5_q-oshwEW6edZ4UvQjN3-tzwi/preview): IMPORTANT pour la compréhension, présentation de l'architecture de l'application et explications des attendus du projet de l'étape 1 à l'étape 5:  
  
- Etape 1: connection sécurisée entre mon niavigateur et serveur web (Nginx) avec Letsenscrypt, MEP firewall, configurer un fail to ban, déployer sur un même serveur les services mail, BdD, pad ;  
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
- boutin.diverse-team.fr associé à walf09 (148.60.11.60) ;  
- boutintest.diverse-team.fr associé à walf10 (148.60.11.139).  
  
# Tâche 1  

Je "fork" le projet sur mon GitHub: https://github.com/WalfroyB/TLC_PROJET_BOUTIN.  
  ![](https://codimd.math.cnrs.fr/uploads/upload_4cf2c9ee61e324eb88c738ac0c596c4e.png)  
  
  
  
  
  
**Notes personnelles**:
  
**Note 01**: Quarkus est un framework d'application Java, conçu pour fonctionner nativement avec Kubernetes, OpenJDK HotSpot et GraalVM. Il offre une faible empreinte mémoire et un temps de démarrage réduit.
  
**Note 02**:  Angular est un framework pour clients, open source, qui permet la création d’applications Web et plus particulièrement d'applications Web monopages : des applications Web accessibles via une page Web unique qui permet de fluidifier l’expérience utilisateur et d’éviter les chargements de pages à chaque nouvelle action.
   
**Note 03**:  
  
**Note 04**:  
    
**Note 05**:  
          
