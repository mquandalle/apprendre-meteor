# Les paquets

Pour avoir du succès, un *framework* - ou plus généralement une *plateforme* - doit réussir à générer un écosystème construit au dessus de lui. Ainsi les succès d'iOS et d'Android sont indissociables de celui des applications mobiles, et le succès de Wordpress indissociable des milliers de plugins disponibles pour l'agrémenter. La gestion des paquets (*packages* en anglais) est donc une problématique majeure pour un framework web comme Meteor ; et fort heureusement le système proposé est particulièrement bien conçu et largement utilisé par une communauté enthousiaste. 

Dans ce chapitre nous allons apprendre à ajouter et supprimer des packages à votre application, puis nous verrons plus en détails les fonctionnalités offertes par certains packages en particulier.

## Présentation du système de paquets

Les packages sont des briques élémentaires apportant des fonctionnalités à une application Meteor. En fait, toutes les fonctionnalités de Meteor sont implémentés dans des packages. Par exemple l'objet `Session` est implémenté dans un package `session`. Les packages peuvent dépendre d'autres packages. Comme nous l'avons vu au chapitre précédent le package `session` dépend du package `deps` qui permet de construire une source réactive.

Pour ajouter un package utilisez la commande : 

```bash
$ meteor add package-name
```

et pour le supprimer utilisez la commande :

```bash
$ meteor remove package-name
```

Ces deux commandes se contentent simplement d'éditer un fichier texte contenant le nom des packages utilisés par l'application. Ce fichier `packages` se trouve dans le répertoire caché `.meteor` situé à la racine de votre projet. Vous pouvez regardez son contenu :

```bash
$ cat .meteor/packages
# Meteor packages used by this project, one per line.
#
# 'meteor add' and 'meteor remove' will edit this file for you,
# but you can also edit it by hand.

standard-app-packages
autopublish
insecure
```

Comme l'indique le commentaire en tête, il est tout à fait possible d'éditer ce fichier à la main sans passer par les commandes `meteor add` et `meteor remove`.

Par défaut notre application utilise trois packages `standard-app-packages`, `autopublish` et `insecure`. En général, une application en production ne devrait pas utiliser `autopublish` et `insecure`. Ces deux packages sont utilisés par défaut uniquement pour accélérer le prototypage d'une application au début de son développement. Nous verrons comment nous en passer dans les prochains chapitres.

`standard-app-packages` est quant à lui un groupe de packages par défaut. Ce package est strictement équivalent à la liste de packages suivants :

```
meteor
webapp
logging
deps
session
livedata
mongo-livedata
templating
handlebars
check
underscore
jquery
random
ejson
```

Il est donc tout à fait possible de remplacer `standard-app-packages` par la liste ci-dessus, ce qui permet éventuellement de la personnaliser. Vous pouvez par exemple supprimer le package `session` si vous n'utilisez pas les variables de session. Ou remplacer `handlebars` par une autre syntaxe de template. Néanmoins il est préférable de garder cette liste inchangée dans un premier temps. Ces packages sont conçus pour fonctionner ensemble et modifier cette liste pourrait introduire des bugs.

L'une des question récurrente posée sur les forums de discussions au sujet du système de package de Meteor concerne le choix d'utiliser un système maison, plutôt que NPM (pour Node Package Manager) le système de Node.js. La réponse est simple : les packages Meteor peuvent faire plus de choses que les packages sur NPM. En particulier ils peuvent fonctionner sur le client, le serveur ou bien les deux. Par exemple, un package de création de formulaires pourra fournir des méthodes pour générer une interface utilisateur côté client, et des méthodes pour la validation des données côté serveur, le tout fonctionnant en harmonie. De plus un package Meteor peut utiliser n'importe quel package NPM comme dépendance côté serveur. Le système développé pour Meteor permet donc de construire des packages fonctionnant à la fois sur le client et sur le serveur tout en bénéficiant de l'écosystème prolifique de Node.js.

## Atmosphere, le dépôt des packages

[TODO] j’attends l'intégration d'atmosphere dans Meteor core

[TODO] capture d'écran atmosphere.

## Quelques packages

### Une API HTTP avec le package `http`

Le package `http` permet à la fois au client et au serveur de réaliser des requêtes HTTP. Comme nous l'avons vu il vous suffit d’exécuter la commande suivante pour ajouter ce package :

```bash
$ meteor add http
```

Une fois ce package ajouté, vous avez accès à un objet global `HTTP` dont la méthode principale s'appelle `call`. Le premier paramètre est la méthode utilisé (`GET`, `POST`, `PUT`, `DELETE`).

[TODO] À détailler

La plupart des packages fonctionnent comme le package `http`, c'est à dire en exportant un objet dans l'espace de nom global (`Deps`, `Session`, etc.). Mais il existe aussi d'autres packages qui modifient la manière de "construire" l'application. C'est le cas par exemple du package `coffeescript`.

### Remplacez JavaScript avec le package `coffeescript`

Dans le premier chapitre d'introduction, nous avions vu que l'un des argument en faveur de Meteor était l'utilisation d'un même langage sur le client et sur le serveur : Javascript.

Pour qui prend le temps de l'étudier, JavaScript est incontestablement un bon langage de programmation. La machine virtuelle V8 développé par Google pour son navigateur Google Chrome, et utilisé par Node.js, est particulièrement performante. Mais le JavaScript souffre aussi d'erreurs de conception qui peuvent être source de bugs. Voyez par exemple les tests d'égalité suivants :

```JavaScript

```

La solution consiste à toujours utiliser l'opérande `===` qui fonctionne comme attendu, en lieu et place de `==`. Pour palier à ces faiblesses certains développeurs choissisent d'utiliser des alternatives à JavaScript comme `Dart` développé par Google, ou `TypeScript` développé par Microsoft. Mais l’alternative la plus populaire à JavaScript s'appelle *CoffeeScript*, qui se définit simplement comme du "sucre syntaxique" pour JavaScript. L'utilisation de coffeescript avec Meteor est incroyablement simple :

```bash
$ meteor add coffeescript
```

Et c'est tout ! Vous pouvez maintenant créer un fichier `myapp.coffee` et développez en CoffeeScript.

[TODO] Présentation rapide de la syntaxe de coffeescript

[TODO] Présentation du source mapping

```coffeescript
Deps.autorun ->
	nbPlayers = Session.get('seleted_players').length
	console.log "#{nbPlayers} joueur(s) séléctionné(s)"
```

### Du CSS amélioré avec le package `stylus`
