# Docker Mode d'Emploi (document provenant Cassiopée Dockerint)

[Dockerint](https://github.com/nordine-marie/dockerint/blob/bfa3deb645940677a2b419b7967a88ee7196dcbd/rep/DockerModeEmploi.md)

# Fonctionnement de Docker 

## Sommaire   

1. Avantage de Docker et description
2. Versions et outils
3. Container 
    * Lancer
    * Rentrer dedans
    * Arreter
    * Supprimer
4. Serveur Nginx
5. Récupérer une image sans la lancer 
6. Afficher l'ensemble contauner existant 
7. Nettoyer système 
8. Créer sa propre image
9. Créer DockerFile
   * Dockerfile
   * Git
   * Construire
10. Créer image DockerHub
11. Docker compose

## Avantage de Docker et description
Le plus : code fonctionne partout, n'isole que ressources nécessaires, démarrage rapide, développement plus autonome, réduites cout, augmente densité de l'infrastructure et améliore cycle déploiement.
Conteneur : isole et le fige dans le temps un seul processus (donc un pour php un pour MySQL etc), on pourra récupérer cet état a tout moment depuis n'importe où et cela fonctionnera. 

## Versions et outils 
Versions Docker : 
Docker Desktop (windows ou mac) a installer et Docker CE gratuit (linux)
Docker entreprise (sous licence)
Les outils : 
Docker Hub : équivalent de github stock image 

## Container
### Lancer
```
docker run nom 
```
*(-d ou --detach si besoin multiple ou garder allumer jusqu'à fin service)*
Docker cherche si "nom" dispo en local sinon sur registry Docker
### Rentrer dedans
```
docker exec -ti ID_RUN_DONNE Bash
cd /use/share/nginx/hotml
```
Modif index.html 
### Arreter
```
docker stop ID_RUN 
```
### Supprimer
```
docker rm ID_RUN
```
## Serveur Nginx

```
docker run -d -p 8080:80 nginx # -p : utilisation des ports
```
En allant sur http://127.0.0.1.8080 page défaut de nginx

## Récupérer une image sans la lancer 
```
docker pull nom
```
## Afficher l'ensemble container existant 
```
docker PS
```
## Afficher l'ensemble des  images 
```
docker images -a
```

## Nettoyer système 
*après bcp tests par exemple (ensemble conteneur, réseau, images, cache suppr)*
```
docker system prune
```

## Créer sa propre image


## Créer Docker file
Dossier ds lequel la recette décrivant l'image dont on a besoin sera écrite avec les diverses dépendances 
Une instruction = un layer = une étape construction de l'image

### Dockerfile
Créer fichier "Dockerfile" 
```
FROM debian:9						//Modèle, Image de base
RUN apt-get update -yq \
&& apt-get install curl gnupg -yq \
&& curl -sL https://deb.nodesource.com/setup_10.x | bash \
&& apt-get install nodejs -yq \
&& apt-get clean -y
ADD . /app/							//télécharger des fichiers ici télécharge source de notre app locale ds /app/
WORKDIR /app						//cd
RUN npm install					//ici installer Node.js
EXPOSE 2368  						//indique port
VOLUME /app/logs    		//indique fichier à partager
CMD npm run start				//commande executer démarrage
```
**Fonctionnement commande CMD et ENTRYPOINT:**
```
CMD ["/bin/service", "-d"]
ENTRYPOINT ["/bin/chamber", "exec", "production", "--"]
```
*Alors les arguments par défaut du conteneur seront ["/bin/chamber","exec", "production", "--","/bin/service", "-d"]
*Ils sont auto-converti en liste.

ATTENTION: Ils peuvent tous deux être des listes, et ENTRYPOINT peut être une liste et CMD peut être une chaîne de caractères; mais si ENTRYPOINT est une chaîne de caractères, CMD sera ignoré --> Toujours faire des listes.

*CMD par defaut: Si on lance Docker run projet avec des arguments, ceux-ci écrasent et remplacent ceux de CMD.
*ENTRYPOINT: Les arguments peuvent aussi écraser ceux de ENTRYPOINT si on a ajouté --entrypoint

Format:
   ```
   docker run --entrypoint argu projet argu
   ```
En général utiliser ENTRYPOINT

### Git
Créer fichier .dockerignore à côté Dockerfile avec
```
node_modules
.git
```

### Construire
```
docker build -t ocr-docker-build .           //. car Dockerfile à la racine
docker run -d -p 2368:2368 ocr-docker-build  //accessible http://127.0.0.1:2368
```

## Créer image DockerHub
Aller sur  https://hub.docker.com/ et se co
Create Repository saisir nom+description
```
docker tag nom:latest ocr/nom:latest                // créer un lien
ou ➜ docker tag id_du_conteneur lieu/nom:latest    //l'id se  trouver avec docker build
docker push YOUR_USERNAME/nom:latest                //envoyer
```

## Docker compose
--> gestion infrastructure, déployer ensemble composant dans conteneurs 
Installation sur Linux sinon inclus 

### Utiliser stack docker compose
```
docker-compose up -d vous permettra de démarrer l'ensemble des conteneurs en arrière-plan ;
docker-compose ps vous permettra de voir le status de l'ensemble de votre stack ;
docker-compose logs -f --tail 5 vous permettra d'afficher les logs de votre stack ;
docker-compose stop vous permettra d'arrêter l'ensemble des services d'une stack ;
docker-compose down vous permettra de détruire l'ensemble des ressources d'une stack ;
docker-compose config vous permettra de valider la syntaxe de votre fichier docker-compose.yml.
```

### Créer docker-compose.yml
Dedans exemple-->
```
version: '3'
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data: {}
```
Lancer:
```
docker-compose up -d
```
Résutat:
http://127.0.0.1:8000

**Fonctionnement commande VOLUME:**
Lorsqu'on surpprime un conteneur tous les éléments créé,modifié sont supprimé. On utilise les volumes pour la persistance des données. Un volume n'augmente pas la taille des conteneurs qui l'utilisent et son contenu existe en dehors du cycle de vie d'un conteneur donné.
```
## Créer une volume
docker volume create <VOLUME NAME>
docker run -ti --name vtest_c -v data-test:/data vtest   //Créer au démarrage -v

# Lister les volumes
docker volume ls

## Supprimer un ou plusieurs volume(s)
docker volume rm <VOLUME NAME>
    -f ou --force : forcer la suppression

## Récolter des informations sur une volume
docker volume inspect <VOLUME NAME>

## Supprimer tous les volumes locaux non inutilisés
docker volume prune
    -f ou --force : forcer la suppression

## Supprimer un conteneur Docker avec le/les volumes associés
docker rm -v <CONTAINER_ID ou CONTAINER_NAME>
    -f ou --force : forcer la suppression
    -v ou --volume : supprime les volumes associés au conteneur
```
