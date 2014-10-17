# Routage

Créer une nouvelle page et faire un lien vers elle est l'une des première chose que l'on apprend dans un cours sur le HTML. La navigation par pages est en effet un concept élémentaire du web depuis sa création. Il peut donc vous sembler étonnant que l'on aborde cette question si tardivement dans ce cours Meteor. C'est qu'entre les débuts du web et aujourd'hui avec Meteor, si les technologies de base sont les mêmes (HTML, CSS, et JavaScript) la manière dont nous les utilisons a fortement changé. Meteor permet de fabriquer des applications dites « monopage », où les actions des utilisateurs (telles qu'un clic sur un bouton) ne rechargent pas une nouvelle page mais se contentent d’exécuter du code JavaScript dans le navigateur. De très nombreuses applications modernes sont bâties sur ce modèle telles que Google Maps, Trello, ou encore EtherPad, et c'est également sur lui que se construisent les applications Meteor. Néanmoins si le concept de pages n'est plus nécessaire pour l'architecture de l'application (qui fonctionne avec une unique page), il est toujours utile de mettre en place un système d'URL pour que vos utilisateurs puissent partager des vues de votre application.

## Encore un routeur

Le rôle d'un routeur est d'associer une URL, ou plus généralement une requête, a une action. Alors qu'il existait déjà un certains nombre de routeurs répondant à ce besoin dans l’écosystème JavaScript, la communauté Meteor s'est rapidement rendue compte de la nécessité d'en concevoir un spécifiquement pour le framework. Les objectifs étaient les suivants :

* Fontionner à la fois sur le serveur et sur le client, et avec la même API (JavaScript isomorphique)
* S'intégrer avec le système de templates, par exemple pour rendre automatiquement un template pour une route donnée
* S'intégrer avec le système de collections, par exemple pour souscrire à un jeu de données pour une route donnée
* Exposer des des méthodes réactives, par exemple pour la route courante

Bien que MDG ait signifié son soutien à un tel routeur, l'entreprise a préféré se reposer sur des initiatives communautaires plutôt que d'imposer un paquet officiel. Si ce positionnement a d'abord entrainé la coexistance de plusieurs solutions concurentes, elles ont depuis fusionné dans un routeur communautaire nommé `iron:router` aujourd'hui largement plébiscité.

## Des routes avec iron:router

Ajouter iron-router s'effectue de manière classique par le système de paquet :

```bash
$ meteor add iron:router
```

Ce paquet expose un objet principal nommé `Router` que vous pouvez utiliser pour créer votre première route.

```js
Router.route('/', function() {
  this.render('home');
});
```

Le premier paramètre contient un chemin, et le second contient l'action a effectuer pour ce chemin – ici afficher le template `'home'`. En fait, si aucune action n'est spécifié, le router rendra automatiquement le template dont le nom correspond au chemin, par exemple :

```js
Router.route('/items');
```

Ici le routeur rendra automatiquement un template nommé `'Items'` ou `'items'` et vous n'avez même pas besoin de spécifier une action.

XXX Trouver une activité à faire... Ajouter des nouvelles routes à l'application todos ?
