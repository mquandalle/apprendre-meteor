# Excursus : outils de développement

Ce chapitre est un *pot pourit* d'outils qui s'avereront utiles lors du développement d'applications Meteor. Aucun de ces outils n'est nécessaire, meteor fonctionne sans aucune dépendance et en particulier le mécanisme de construction de l'application est directement intégré à l'utilitaire en ligne de commande et ne nécessite pas d'utiliser un outil tiers du type `grunt`.

Libre à vous d'explorer le fonctionnement de ces différents outils avant d'entammer le TP qui cloturerat cette première partie.

## Gestion de version

Meteor ne vous impose, ni n'impose aux auteurs de paquets, aucun logiciel de gestion des sources en particulier. Néanmoins, une écrasante majorité de sa communauté utilise `git`.

## Débogage

Le débogage d'une application Meteor ne diffère pas fondamentalement des méthodes normales pour les applications web. Meteor propose néanmoins certaines commandes utiles que nous présentons dans cette section.

### Outils de débogage

Utilisation d'un point d'arret pour rentrer dans le contexte d'un fichier ← exemple avec MeteorPad
Async ?

Débogage côté serveur (node-inspector) ← utiliser la nouvelle commande

Possibiliter de sauvegarder les fichiers directement depuis le déboggeur ?

### Logs

Vous pouvez accéder aux logs du serveur (le texte affiché *via* la méthode `console.log`) avec la commande `meteor logs www.mondomaine.com`. Si vous avez défini un mot de passe, il vous sera alors demandé.

### Sources maps

Source maps c'est pratique, utilisé par exemple par coffeescript, mais pas seulement

> Pas de source maps pour les templates pour l'instant

## MongoDB

La commande `meteor mongo` vous donne accès à un shell interactif pour interagir avec la base de données de votre application. Pour le moment il faut que l'application soit déjà lancée.

> `meteor mongo` fonctionne également avec les application déployées dans le nuage Meteor. Utilisez simplement `meteor mongo monappli.com`. Si vous avez défini un mot de passe il vous sera demandé.

meteor mongo --url

Robomongo

@TODO capture d'écran de robomogo

XXX Pas beaucoup de MAJ de robomongo ces derniers temps, devrai-je présenter une autre UI?

du coup

MONGO_URL=$(meteor mongo --url http://<your-app>.meteor.com/) meteor

voir aussi meteor reset

## Éditeurs

Discussions IDE vs éditeur de texte

Pour les éditeurs plugins vim, sublime text, atom

Pour les IDE webstorm (gratuit pour une période d'essai de 30j)

---

À l'heure actuelle il n'existe pas d'outil de déboggage qui fonctionne pour plusieurs plateformes à la fois (client, serveur, et cordova par exemple) et nous présenterons donc ces outils dans des sections distinctes. Il existe cependant une initiative de Mozilla pour standariser le protocole utilisé par les outils de débogage et permettre de begoger plusieurs plateforme depuis la même interface.