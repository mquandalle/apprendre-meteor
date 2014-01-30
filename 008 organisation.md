# Excursus: Organisation du code

Une application Meteor est la somme du code JavaScript exécuté dans le navigateur, du JavaScript exécuté dans le processus Node.js ainsi que de quelques ressources statiques comme les styles CSS et les images. Le packaging et la transmission de ces éléments sont automatisés et suivent des règles que nous allons présenter dans ce chapitre.

## Arborescence

Contrairement à d'autre framework web, Meteor est très flexible quant à l'organisation de l'arborescence d'une application. Comme nous l'avons vu jusqu'ici, par défaut un fichier quelquonque est exécuté à la fois sur le client et sur le serveur. Il existe cependant quelques règles particulières.

### Code client et code serveur

* un fichier dans un répertoire `/client` sera exécuté uniquement du côté client
* un fichier dans un répertoire `/server` sera exécuté uniquement du côté serveur

Pour des applications de taille moyenne ou importante, on préféra donc utiliser ces répertoires plutôt que les conditions `Meteor.isClient` et `Meteor.isServer`. Notez que les répertoires `client` et `server` peuvent être des sous répertoires, ce qui permet d'organiser son projet avec des modules de premier niveau, par exemple `/blog` et `/forum` à l'intérieur desquelles se trouvent les répertoire `client` et `server`.

### Ressources publiques

Les ressources publiques telles que les images, un sitemap ou un fichier `robots.txt` doivent être placer dans le répertoire `/public` à la racine du projet.

Requetable 

> Pour des grosses applications mettre Nginx comme proxy plus performant

### Les `Assets` des ressources privées

l'API Assets

### Ordre du code JavaScript

Main.js en premier
Lib en premier

## Espaces de noms

Meteor ajoute des closures sur tous les fichiers. Interet.
compatibilites

## Production

