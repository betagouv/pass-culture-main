# pass-culture-main

C'est tout le framework du Pass Culture!

## Minimal Process

### Install

Il vous faudra une machine UNIX.

Installer:
- [docker](https://docs.docker.com/install/)
- [docker-compose](https://docs.docker.com/compose/install/#install-compose)
- [yarn](https://yarnpkg.com/fr/) voir le README dans le dépot https://github.com/betagouv/pass-culture-browser/

Mais spécialement, en plus pour macosx:
- brew install coreutils


Enfin pour tout le monde:

```bash
./pc symlink
```
puis

```bash
pc install
```

### Init
Pour verifier les tests:
```bash
pc test-backend
```

Pour avoir une database de jeu:
```bash
pc sandbox -n industrial
```

### Démarrage

Pour lancer l'API:
```bash
pc start-backend
```

Pour lancer l'appli Web:
```bash
pc start-webapp
```

Pour lancer le portail pro:
```bash
pc start-pro
```


## Développeurs.ses

### Rebuild

Pour reconstruire l'image docker sans cache
```bash
pc rebuild-backend
```

### Restart

Pour effacer la base de données complétement, et relancer tous les containers:
```bash
pc restart-backend
```

### Reset

Si vos serveurs de dev tournent, et que vous souhaitez juste effacer les tables de la db:
```bash
pc reset-sandbox-db
```

Si vous voulez juste enlever les recommandations et bookings crées en dev par votre navigation:
```bash
pc reset-reco-db
```


### Migrate

Vous pouvez passer toutes les cli classiques d'alembic comme ceci:
```bash
pc alembic upgrade
```

### Test

Pour tester les apis du backend:
```bash
pc test-backend
```

Pour tester les apis du frontend:
```bash
pc test-frontend
```

Pour tester la navigation du site web
```bash
pc -e production test-cafe-webapp -b firefox
```

Exemple d'une commande test en dev sur chrome pour un fichier test particulier:
```bash
pc test-cafe-pro -b chrome:headless -f signup.js
```

### Restore DB

Pour restorer un fichier de dump postgresql (file.pgdump) en local:

```bash
pc restore-db file.pgdump
```

Pour anonymiser les données après restauration, et changer le mot de passe pour tout les users :

```bash
./anonymize_database.sh -p PASSWORD
```

### Database de jeu

Afin de naviguer/tester différentes situations de données, il existe dans api des scripts permettant d'engendrer des bases de données "sandbox".

La plus conséquente est l'industrial, elle se crée via la commande:

```bash
pc sandbox -n industrial
```

Cette commande faite, il y a alors deux moyens pour avoir les email/mots de passe des utilisateurs sandbox :

  - on peut utiliser la commande sandbox_to_testcafe qui résume les objets utilisés de la sandbox dans les différents testcafés. Si on veut avoir tous les utilisateurs des tests pro_07_offer dans l'application pro, il faut faire:
  ```
    pc sandbox_to_testcafe -n pro_07_offer
  ```
  - on peut utiliser un curl (ou postman) qui ping directement le server à l'url du getter que l'on souhaite:
  ```
  curl -H "Content-Type: application/json" \
       -H "Origin: http://localhost:3000" \
       GET http://localhost:80/sandboxes/pro_07_offer/get_existing_pro_validated_user_with_validated_offerer_validated_user_offerer_with_physical_venue
  ```

Il est important que votre server api local tourne.

Pour les développeur.ses, quand vous écrivez un testcafé,
il faut donc la plupart du temps écrire aussi un getter côté api dans
sandboxes/scripts/getters/<moduleNameAvecleMêmeNomQueLeFichierTestcafe>, afin
de récupérer les objets souhaités dans la sandbox.

Si vous voulez avoir un aperçu global de l'application webapp, vous pouvez tout de même vous connecter avec cet user toujours présent:

```
email: pctest.jeune93.has-booked-some@btmx.fr
password: pctest0.Jeune93.has-booked-some
```

Pour l'application PRO, vous pouvez naviguer en tant qu'admin avec:

```
email: pctest.admin93.0@btmx.fr
password: pctest0.Admin93.0
```

Ou en tant qu'user avec :

```
email: pctest.pro93.0@btmx.fr
password: pctest.pro93.0
```

(Ces deux utilisateurs existent également pour le 97, pour les utiliser, il suffit de remplacer 93 par 97)

## Tagging des versions

La politique de tagging de versions est la suivante :
* on n'utilise pas de _semantic versioning_
* on utilise le format `I.P.S`
  * I => incrément d'__Itération__
  * P => incrément de _fix_ en __Production__
  * S => incrément de _fix_ en __Staging__

### Exemple

* Je livre une nouvelle version en staging en fin d'itération n°20 => `20.0.0`
* Je m'aperçois qu'il y a un bug en staging => `20.0.1`
* Le bug est corrigé, je livre en production => `20.0.1`
* On détecte un bug en production, je livre en staging => `20.1.0`
* Tout se passe bien en staging, je livre en production => `20.1.0`
* On détecte un autre bug en production, je livre en staging => `20.2.0`
* Je m'aperçois que mon fix est lui-même buggé, je relivre un fix en staging => `20.2.1`
* Mes deux fix sont cette fois OK, je livre en production => `20.2.1`

## Deploy

### FRONTEND WEB

Pour déployer une nouvelle version web, par exemple en staging:
**(Attention de ne pas déployer sur la production sans concertation !)**

```bash
pc -t I.P.S tag-version-webapp
pc -e staging -t I.P.S deploy-frontend-webapp
```

Et pour pro sur staging, il suffit de remplacer la dernière commande par celle-ci :
```bash
pc -t I.P.S tag-version-pro
pc -e staging -t I.P.S deploy-frontend-pro
```

Pour déployer en production ensuite :
```bash
pc -e production -t I.P.S deploy-frontend-webapp
```

#### Publier shared sur npm

Pour publier une version de pass-culture-shared sur npm

```bash
cd shared
npm adduser
yarn version
yarn install
npm publish
```

Puis sur webapp et/ou pro, mettre à jour la version de pass-culture-shared dans le fichier `package.json` :

```bash
yarn add pass-culture-shared@x.x.x
git add package.json yarn.lock
```

avec `x.x.x` nouvelle version déployée sur shared.


### BACKEND

Pour déployer une nouvelle version de l'API, par exemple en staging:
**(Attention de ne pas déployer sur la prod sans autorisation !)**

```bash
pc -t I.P.S tag-version-backend
pc -e staging -t I.P.S deploy-backend
```

Avec vI.P.S, le tag de la version à déployer.
Ensuite pour mettre en production le tag qui a été déployé sur l'environnement staging :

```bash
pc -e production -t I.P.S deploy-backend
```



## Administration

### Connexion à la base postgreSQL d'un environnement

Par exemple, pour l'environnement staging :

```bash
pc -e staging psql
```

### Connexion à en ligne de commande python à un environnement

Par exemple, pour l'environnement staging :

```bash
pc -e staging python
```

Il est également possible d'uploader un fichier dans l'environnement temporaire grâce à la commande suivante :

```bash
pc -e staging -f myfile.extension python
```

L'option -f est également disponible pour la commande bash :

```bash
pc -e staging -f myfile.extension bash
```


### Gestion des objects storage OVH

Pour toutes les commandes suivantes, vous devez disposer des secrets de connexion.


Pour lister le contenu d'un conteneur sépcific :

```bash
pc list_content --container=storage-pc-staging
```

Pour savoir si une image existe au sein d'un conteneur :

```bash
pc does_file_exist --container=storage-pc-staging --file="thumbs/venues/SM"
```

Pour supprimer une image au sein d'un conteneur :

```bash
pc delete_file --container=storage-pc-staging --file="thumbs/venues/SM"
```

Pour faire un backup de tous les fichiers du conteneur de production vers un dossier local :

```bash
pc backup_prod_object_storage --container=storage-pc --folder="~/backup-images-prod"
```

Pour copier tous les fichiers du conteneur de production vers le conteneur d'un autre environnement :

```bash
pc copy_prod_container_content_to_dest_container --container=storage-pc-staging
```


## Gestion OVH

#### CREDENTIALS

Vérifier déjà que l'un des admins (comme @arnoo) a enregistré votre adresse ip FIXE (comment savoir son adress ip? http://www.whatsmyip.org/)

#### Se connecter à la machine OVH

```bash
pc -e production ssh
```

### Dump Prod To Staging

ssh to the prod server
```bash
cd ~/pass-culture-main && pc dump-prod-db-to-staging
```

Then connect to the staging server:
```bash
cd ~/pass-culture-main
cat "../dumps_prod/2018_<TBD>_<TBD>.pgdump" docker exec -i docker ps | grep postgres | cut -d" " -f 1 pg_restore -d pass_culture -U pass_culture -c -vvvv
pc update-db
pc sandbox --name=webapp
```


### Updater le dossier private

Renseigner la variable d'environnement PC_GPG_PRIVATE.
Puis lancer la commande suivante :

```bash
pc install-private
```

#### Updater la db
Une fois connecté:
```
cd /home/deploy/pass-culture-main/ && pc update-db
```

#### Note pour une premiere configuration HTTPS (pour un premier build)

Pour obtenir un certificat et le mettre dans le nginx (remplacer <domaine> par le domaine souhaité, qui doit pointer vers la machine hébergeant les docker)
```bash
docker run -it --rm -v ~/pass-culture-main/certs:/etc/letsencrypt -v ~/pass-culture-main/certs-data:/data/letsencrypt deliverous/certbot certonly --verbose --webroot --webroot-path=/data/letsencrypt -d <domaine>
```

Puis mettre dans le crontab pour le renouvellement :
```bash
docker run -it --rm -v ~/pass-culture-main/certs:/etc/letsencrypt -v ~/pass-culture-main/certs-data:/data/letsencrypt deliverous/certbot renew --verbose --webroot --webroot-path=/data/letsencrypt
```





## Version mobile (outdated, but can be useful someday)

### Emuler avec Cordova

```bash
yarn global add cordova-cli
```

```bash
cd webapp && cordova run ios
```

<!--
iPhone-5s, 11.2
iPhone-6, 11.2
iPhone-6-Plus, 11.2
iPhone-6s, 11.2
iPhone-6s-Plus, 11.2
iPhone-7, 11.2
iPhone-7-Plus, 11.2
iPhone-SE, 11.2
iPad-Air, 11.2
iPad-Air-2, 11.2
iPad--5th-generation-, 11.2
iPad-Pro--12-9-inch---2nd-generation-, 11.2
iPad-Pro--10-5-inch-, 11.2
Apple-TV-1080p, tvOS 11.2
Apple-TV-4K-4K, tvOS 11.2
Apple-TV-4K-1080p, tvOS 11.2
iPhone-8, 11.2
iPhone-8-Plus, 11.2
iPhone-X, 11.2
iPad-Pro--9-7-inch-, 11.2
iPad-Pro, 11.2
Apple-Watch-38mm, watchOS 4.2
Apple-Watch-42mm, watchOS 4.2
Apple-Watch-Series-2-38mm, watchOS 4.2
Apple-Watch-Series-2-42mm, watchOS 4.2
Apple-Watch-Series-3-38mm, watchOS 4.2
Apple-Watch-Series-3-42mm, watchOS 4.2
-->

### Développer en Android

Vous pouvez utiliser une ptite config ngrok pour l'api et la webapp par exemple:
```bash
cd webapp/ && yarn run ngrok
```

Ensuite il faut lancer l'application configurée avec ces tunnels:
```bash
pc start-browser-webapp -t
```

Vous pourrez alors utiliser l'url ngrok webapp pour dans votre navigateur android.

### Déployer le FRONTEND MOBILE

Pour déployer une nouvelle version phonegap (par default c'est en staging)
```bash
pc build-pg
```

## Lancer les tests de performance

### Environnement

Les tests requièrent d'avoir un environnement spécifique sur Scalingo, ici `pass-culture-dev-perf`, comportant notamment une base utilisateur.
Pour la remplir, il faut jouer les sandboxes `industrial` et `activation`.

Execution des sandboxes sur le conteneur :
``` bash
scalingo -a pass-culture-dev-perf run 'PYTHONPATH=. python scripts/pc.py sandbox -n industrial'
scalingo -a pass-culture-dev-perf run 'PYTHONPATH=. python scripts/pc.py sandbox -n activation'
```

Ensuite, lancer le script d'import des utilisateurs avec une liste d'utilisateurs en csv prédéfinie placée dans le dossier `artillery` sous le nom `user_list`.
On passe en paramètre un faux email qui ne sera pas utilisé.

``` bash
scalingo -a pass-culture-dev-perf run 'ACTIVATION_USER_RECIPIENTS=<email> python scalingo/import_users.py user_list' -f user_list

```

Un exemple de csv utilisateur `user_list` :
```bash
1709,Patricia,Chadwick,ac@enimo.com,0155819967,Drancy (93),16282,2001-05-17,secure_password
```

### Lancement d'un scénario

Pour lancer les tests de performance il faut installer le logiciel `artillery` : `npm install -g artillery` et son plugin `metrics-by-endpoint` : `npm install artillery-plugin-statsd`, puis se munir du fichier csv contenant
les users valides.

Puis se placer dans le dossier `artillery` et lancer la commande :

```bash
artillery run scenario.yml -o reports/report-$(date -u +"%Y-%m-%dT%H:%M:%SZ").json
```

Un rapport des tests daté sera généré dans le sous-dossier `reports` (qui doit être crée).
