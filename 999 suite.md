# Excursus : Meteor hors ligne

- Principe
- appcache
- les données
- Phonegap + Meteor

# Les méthodes

Les méthodes sont la dernière notion de base qu'il nous reste à aborder. En fait, nous avons déjà utilisé les méthodes via l'utilisation des collections qui sont implémentées en utilisant les méthodes

## Présentation des méthodes

Une méthode est une fonction éxecutée sur le serveur que le client peut appeler. 

```
Meteor.methods({
	updateTwitterFeed: function(twitterId) {
		console.log('');
	}
});
```

## Compensation de latence

## Sécurité

Match et check
audit-arguments-check

\part{Packages populaires}

# Des tests avec Laika

# Routing avec Iron-Router

# Collection2, SimpleSchema et AutoForm

\part{Notions avancées}

> "Dans notre effort pour comprendre la réalité, nous ressemblons un peu à un homme qui tente de comprendre le mécanisme d’une montre fermée. Il voit le cadran et les aiguilles, il entend même le tic-tac, mais n’a aucun moyen d’ouvrir le boîtier. " -- A. Einstein

Meteor permet à des débutants d'être rapidement productifs dans le développement d'application web complexe. Mais le framework peut parfois sembler "magique". Cette deuxième partie vous permettra de comprendre les mécanismes internes les plus importants. Ces connaissances vous permettrons ensuite de créer vos propores sources réactive, votre propre package on votre propre type personnalisé pour EJSON.

Contrairement à la première partie consacrée à l'apprentisage des bases, une lecture linéaire n'est pas nécessaire pour cette deuxième partie. Les chapitres sont indépendants entre eux et vous pouvez étudier un sujet quant bon vous semble.

# Créer un package

## presentation de l'API

## comment publier son package sur atmosphere

# EJSON

introduction sur le json

## Interet de Ejson

Date et binary

## Créer vos propres types personnalisés

# Meteor UI

# DDP

- Présentation de DDP
  - Principe séparation du code application et des données (qui sont servies par DDP)
  - Basé sur Websocket si disponible fallback HTTP sinon
- Étude des messages envoyés lors d'une souscription, d'une requête
- Utiliser un client DDP pour une extension chrome

# Fibers, future
- presentation programmation asynchrome, probleme des callback et du code spaghetti
- bindenvironnement
- c'est pour ça qu'il y a Meteor.setTimeout
cf eventedmind

# Scalabilité
- organisation des différents serveurs utilisé dans une appli meteor

\part{Annexes}

# Sécurité
- intro: annexe qui récapitule tous les élements concernant la sécurité d'une application Meteor
  principe de base de sécurité, on ne croit jamais ce que le client fait
- check, Match
- insecure, allow deny
- browser-policy

# Bien utiliser MongoDB

## robomongo

## denormalisation

## l'API MongoDB

# Les ressources pour aller plus loin
- la documentation des différentes technologies: meteor, mongodb, handlebars
- les forums officiels google groupes
- irc
- les sites avancés, eventedmind
- lire le code!
- crater.io, meteorhacks, meteorjobs

# Glossaire
- Une liste de notions/vocabulaire utilisé dans le monde Meteor