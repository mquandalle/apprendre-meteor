# Excursus : Déploiement

Ce cours est parsemé de quelques chapitres excursus dont la lecture n'est pas requise pour comprendre le reste du cours. Celui-ci concerne le déploiement d'une application Meteor. Si vous voulez améliorer votre leaderboard avant de le montrer au monde entier, vous pouvez passer ce chapitre et y revenir plus tard.

Ce chapitre présente deux possibilités pour la mise en production d'une application: le nuage Meteor et votre propre serveur dédié. Nous présenterons ensuite les principaux concepts mis à l'œuvre lors du déploiement d'une application Meteor.

## Déploiement dans le nuage Meteor

*Meteor Developpement Group* dite *MDG* est la société principale contributrice du projet Meteor. Elle met à disposition un hébergement dans le nuage, pour le moment gratuit. Y déployer son application est d'une simplicité enfantine :

```bash
$ meteor deploy monsousdomaine.meteor.com
```

Dans la commande ci-dessus vous devez évidemment remplacer `monsousdomaine` par un nouveau nom de sous domaine qui n'est pas encore utilisé. Une fois l'opération terminée, rendez-vous sur <http://monsousdomaine.meteor.com> et voilà ! Votre application est maintenant publique et le monde entier peut y accéder.

Si c'est la première fois que vous déployer une application, on vous demandera de créer un compte développeur Meteor. Si vous avez déjà un compte vous pouvez vous connecter avec la commande `meteor login`. Ce compte vous permettra de gérer les applications déployées sur le nuage Meteor.

> Vous pouvez également utiliser votre propre nom de domaine. Il faut pour cela définir un enregistrement DNS `CNAME` redirigeant vers `origin.meteor.com`. Vous pourrez ensuite déployer avec la commande précédente :
>
> ```bash
> $ meteor deploy www.mondomaine.com
> ```

Pour mettre à jour votre application, il vous suffit de développer les modifications en local, puis d’exécuter à nouveau la commande `meteor deploy`. Pour la supprimer utiliser l'option `--delete` avec la commande précédente.

Cette solution d’hébergement est gratuite pour le moment, mais MDG devrait proposer une offre commerciale dans les prochains mois.

## Déploiement sur votre propre serveur

MDG proposant de déployer nos applications sur leur serveurs, cela veut-il dire que je suis enfermé dans une prison, fût-elle aux barreaux d'or ? Absolument pas. D'une part la licence MIT utilisée par le framework vous permet d'en faire absolument se que vous voulez, et d'autre part l'utilitaire en ligne de commande vous permet de construire facilement une application Node.js indépendante.

Si vous n'avez pas l'intention dans l'immédiat d’héberger une application Meteor sur votre propre serveur, vous pouvez passer cette section sans problème.

### Construction

La commande en question s'appelle `meteor build`. Elle permet de générer une fabriquer une application indépendante pour chaque plateforme supporté par l'application, nous utiliserons ici l'application Node.js mais la commande permet aussi de générer un fichier `apk` pour Android ou une application pour iOS comme nous le verrons dans le chapitre consacré aux applications mobiles. En fait cette commande est utilisée sous le capot lorsque vous développez votre application avec un `meteor run` ou lorsque vous la déployez avec un `meteor deploy`.

```bash
$ meteor build --directory /tmp/app/
$ cd /tmp/app/
$ cat README
This is a Meteor application bundle. It has only one external dependency:
Node.js 0.10.29 or newer. To run the application:

  $ (cd programs/server && npm install)
  $ export MONGO_URL='mongodb://user:password@host:port/databasename'
  $ export ROOT_URL='http://example.com'
  $ export MAIL_URL='smtp://user:password@mailhost:port/'
  $ node main.js

Use the PORT environment variable to set the port where the
application will listen. The default is 80, but that will require
root on most systems.

Find out more about Meteor at meteor.com.
```

Toute soucieuse qu'elle est de vous faciliter la vie, la commande `build` génère un fichier `README` qui contient les instructions pour lancer l'application sur un serveur de production. Comme indiqué dans ce fichier la seule dépendance à installer sur le serveur est nodejs. Il faut ensuite se rendre dans le répertoire contenant l'application serveur et y installer les dépendances NPM  – NPM est le système de paquets fourni avec nodejs. Puis il faut définir quelques variables d'environnement pour indiquer à l'application les identifiants de connexion à la base de donnée ou au serveur SMTP pour envoyer des emails aux utilisateurs. Enfin on lance l'application avec `node main.js`. Meteor a généré une application nodejs normale qui fonctionne donc dans tous les environnement supportant nodejs et il n'est pas nécessaire de télécharger l'utilitaire `meteor` sur le serveur de production. En conséquence l'application fonctionnera avec les plateformes d’hébergement dans le nuage supportant nodejs comme [heroku](https://www.heroku.com/), [nodejitsu](https://www.nodejitsu.com/), ou le français [clevercloud](https://www.clever-cloud.com/fr/).

Si vous préférez déployer l'application sur votre un serveur dédié, vous aurez naturellement à en assurer l'exploitation comme pour n'importe quelle application nodejs. En particulier il peut-être utile d'installer de mettre en place un système de cache en utilisant un proxy comme [nginx](http://nginx.org/), [apache](http://httpd.apache.org/), ou [HAProxy](http://www.haproxy.org/) pour servir les ressources statiques de manière performante. Il faut également s'assurer que l'application est démarrée automatiquement en cas de redémarrage du serveur , et que le processus sera automatiquement relancé s'il venait à crasher (en utilisant par exemple [forever](https://github.com/nodejitsu/forever) ou [pm2](https://github.com/Unitech/pm2).

### meteor up

Pour faciliter l'exploitation et la mise en production de nouvelles versions de votre application sur un serveur dédié Debian ou Ubuntu, Arunoda Susiripala, un membre éminent de la communauté Meteor, a crée un utilitaire qui permet d'automatiser ces taches. Le projet est libre, s'appelle [meteor up](https://github.com/arunoda/meteor-up), et s'installe en utilisant `npm` :

```bash
$ npm install -g mup
```

Meteor up doit être installé à la fois sur le serveur et sur le client. Ses fonctionnalités incluent :

* déploiement en une seule ligne de commande
* gestion des configurations et des variables d'environnement
* démarrage automatique de l'application en cas de reboot du serveur
* redémarrage automatique de l'application en cas de crash du process nodejs
* installation automatique de MongoDB sur le serveur
* déploiement sur plusieurs serveurs d'une même application

L'utilisation de meteorup n'est pas au programme de ce cours, si vous êtes intéressé par cet utilitaire la meilleure ressource pour apprendre son utilisation et ses options reste la [documentation officielle](https://github.com/arunoda/meteor-up/blob/master/README.md).


## Concepts

### Minification

En production il convient d'optimiser les ressources que nous servons aux clients. Parmi les courantes on trouve la fusion de tous les fichiers JavaScript (respectivement CSS) pour minimiser le nombre de requête HTTP nécessaires, ou encore la « minification » du code qui consiste à réduire le volume des ressources en utilisant des optimisations comme le renommage des variables.

L'utilisation de `meteor deploy` ou de `meteor build` s'occupe automatiquement de « minifier » toutes les ressources de votre application de manière transparente. Votre application sera donc optimisée en production s'en que vous n'ailliez rien à faire !

> Il peut exister quelques rares cas, des bogues, pour lesquels le comportement de l'application minifié diffère de celui de l'application durant le développement. Si vous suspecter votre application de contenir un tel bogue, vous pouvez demander utiliser `meteor run --production` qui permet de simuler le mode de production.

### Reproductibilité

Meteor assure la reproductibilité de la construction des applications. Cela veut dire que pour une application donnée, Meteor assure que la construction sur une machine donnée fabrique le même résultat qu'une construction effectuée sur une autre machine à partir du même code source. Comme nous le verrons dans le chapitre consacré au système de paquets, Meteor utilise pour se faire un fichier contenant l'ensemble des versions des dépendances utilisées par l'application.

C'est grâce au respect de ce principe de reproductibilité qu'il n'est pas nécessaire d'installer Meteor sur un serveur en production. On peut se contenter de construire l'application une seule fois et la déployer sur autant de plateformes que désiré.

### Configuration et paramètres
↑ Déplacer vers « Notions avancées » ?

La configuration d'une application en production s'effectue avec des variables d'environnement sur le serveur. C'est une décision de conception conséquence de la reproductibilité présentée à la section précédente. Il faut en effet que le puisse changer la configuration d'une application sans avoir à la re-construire. Ce modèle rend par exemple possible la distribution d'une application Node.js que chaque utilisateur peut ensuite installer sur son propre serveur et configurer pour utiliser sa propre base de donnée.

Bien qu'elles ne soient pas toutes documentées, voici les variables d'environnement les plus utiles :

* `ROOT_URL` : URL publique de l'application [requis en production]
* `MONGO_URL` : URI vers la base MongoDB [requis en production]
* `EMAIL_URL` : URI vers un serveur SMTP [requis pour envoyer des emails]
* `DISABLE_WEBSOCKETS` : Choisir la valeur `1` pour désactiver l'utilisation des websockets
* `BIND_IP` : l’adresse IP sur laquelle écouter (utile si le serveur possède plusieurs addresses)
* `PORT` : le port sur lequel écouter (par défaut 3000). Écouter sur un port inférieur à 1024 requiert en général les privilèges de super-utilisateur.

L'utilitaire meteor up vous permet de définir ces variables d'environnement directement dans un fichier `.json` de configuration ce qui facile la tâche pour certains environnements.

@TODO `Meteor.settings`
