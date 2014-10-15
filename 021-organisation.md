# Excursus: Organisation du code

Meteor laisse une grande liberté quant à l'organisation du code source d'une application. Peut-être même trop grande. Car contrairement aux principaux autres frameworks, Meteor n'impose pas de structure particulière, il est par exemple possible de centraliser tout le code dans un seul fichier (comme nous l'avons vu avec l'application leaderboard), ou bien de n'écrire qu'une seule fonction par fichier, ou bien d'enfouir un fichier dans une longue arborescence de sous-répertoires. Mises à part quelques répertoires particuliers pour lesquels Meteor a un comportement par défaut, le framework se contente en général d'assembler tous les fichiers qu'il trouve dans l'arborescence sans qu'on lui indique quoi que se soit. Si le développeur n'y prend garde, ce comportement pourra résulter dans une application sans structure ou l'on ne saura plus où donner de la tête.

Ce chapitre, en plus d'une présentation exhaustive des quelques règles d'organisation imposées par Meteor, proposera donc un ensemble de conventions d'organisation. Naturellement ces conventions ne sont que des conseils qui pourront être adaptés pour les besoins spécifiques de votre application.

## Répertoires particuliers

Comme nous l'avons vu jusqu'ici, par défaut un fichier quelconque est exécuté à la fois sur le client et sur le serveur. Il existe cependant cinq répertoires particuliers dérogeant à cette règle.

### Code client et code serveur

Puisqu'il est courant d'écrire du code spécifique au client, ou spécifique au serveur, Meteor respecte les règles suivantes :

* Les fichiers présents dans un répertoire `client/` seront exécutés uniquement du côté client
* Les fichiers présents dans un répertoire `server/` seront exécutés uniquement du côté serveur

Pour des applications de taille moyenne ou importante, on préféra toujours utiliser ces répertoires plutôt que les conditions `Meteor.isClient` et `Meteor.isServer`. Typiquement, les règles `allow` et `deny` qui ne sont définies que sur le serveur pourront être placées dans un fichier `server/authorisations.js`. Le code lié à la gestions des templates, des événements, et de la réactivité sera quant à lui placé dans le répertoire `client` puisqu'il concerne des concepts valables uniquement dans le navigateur.

```
├── client
│   ├── leaderboard.html
│   ├── leaderboard.js
│   └── leaderboard.css
└── server
    └── authorisations.js
```

### Ressources statiques

Les ressources statiques publiques sont des fichiers qui doivent être servi brut, c'est à dire sans modification ni de son contenu ni de son nom. Il s'agit par exemple du fichier `robots.txt` qui est utilisé pour communiquer avec les robots d'indexation des moteurs de recherche, ou bien encore de la plupart des images telles qu'un logo ou que les icônes utilisées par l'application. Ces ressources doivent être placées dans le répertoire `/public/` au niveau de la racine de l'application :

```
├── client
├── public
│   ├── followIcon.png
│   ├── logo.png
│   └── robots.txt
└── server
```

Les fichiers présents dans le répertoire `/public/` sont servis en réponse à une requête HTTP GET dont l'URL d’interrogation correspond à leur nom. Par exemple dans le cas si dessus, je peux lire le fichier `robots.txt` à l'URL <http://localhost:3000/robots.txt>.

> Pour des applications devant soutenir un trafic important, il convient d’intégrer un serveur proxy comme `nginx` ou `HAProxy` en amont du serveur Meteor. Ces serveurs sont en effet plus rapides et ils permettent des options avancées pour la gestion du cache ou la protection contre les attaques par déni de service.

Même s'il elles sont en général utilisées plus rarement que les ressources publiques, il est également possible de définir des ressources statiques privées dans un répertoire `/private/` :

```
├── client
├── private
│   ├── data.txt
│   └── secret.bin
├── public
└── server
```

Ces ressources ne seront pas accessibles aux clients mais uniquement au serveur. Pour lire une ressource privée, on utilise l'objet `Assets` défini uniquement du côté serveur :

```javascript
var data = Assets.getText('data.txt');
var secretBin = Assets.getBinary('secret.bin');
```

Les méthodes `getText` et `getBinary` s'utilisent simplement en indiquant le chemin vers la ressource privée désirée (relativement au répertoire `/private/`). `getText` retourne la ressource interprété comme du texte `UTF-8`.

### Tests

En plus des répertoires `client`, `server`, `public`, et `private`, il existe un cinquième et dernier répertoire au fonctionnement particulier : `tests`. Comme son nom l'indique ce répertoire contient le code dédié au tests de l'application. Nous l’étudierons dans un chapitre qui y est consacré.

```
├── client
├── private
├── public
├── server
└── tests
```

Le code placé à l’extérieur de ces cinq répertoires sera exécuté à la fois sur le client et sur le serveur.

## Espaces de noms

Les espaces de noms ne sont certainement pas la notion la plus courante dans l'écosystème JavaScript. Et pour cause le langage offre des possibilités un peu limitées en la matière. Pourtant au fur et à mesure qu'une application grandit, et qu'elle inclut notamment du code tiers via le système de paquets, le risque d'effets de bords liés à une collision de noms s'accroit d'autant plus. Avant d'exposer la manière dont Meteor traite ce problème, voici quelques petit rappels sur la portée des variables en JavaScript.

Il existe deux portées de variable possible en JavaScript : locale ou globale. Les variables globales sont déclarées à l’extérieur d'une fonction. Les variables locales sont déclarées à l’intérieur d'une fonction et sont locales à cette fonction.

```javascript
var a = "variable globale";

var b = function() {
  var c = "variable c locale à la fonction b";
  return "la fonction b est une fonction globale";
};

var x = 5;
var f = function(y){
  var z = y + 1;
  return x + z;
};
console.assert(f(4) === 10);
x = 0;
console.assert(f(4) === 5);
```

Jusqu'ici rien de bien compliqué. Il existe cependant une subtilité de JavaScript qui s'il est méconnue risque de causer de bogues importants : une variable déclarée sans le mot clé `var` est une variable globale (même s'il est définie à l'intérieur d'une fonction).

Ainsi :

```javascript
var x = 5;
var f = function() {
  // Danger, on modifie la variable globale, et non une variable locale
  x = 10;
}
console.assert(x === 5);
f();
console.assert(x !== 5);
```

Puisque les risques d'effets de bord proviennent de l'utilisation de variables globales au lieu de locales, il faut toujours utiliser le mot clé `var` pour déclarer une variable (sauf s'il l'on est sûr de ce que l'ont fait). C'est d'ailleurs pour éviter ce type d'erreurs que des précompilateurs comme CoffeeScript ajoutent systématiquement le mot clé `var` :

```coffeescript
x = 5
f = ->
  # Avec CoffeeScript l'assignation suivante est locale,
  # et ne vient pas polluer l'espace global
  x = 10

console.assert(x == 5)
f()
console.assert(x == 5)
```

L'autre risque de pollution involontaire de l'environnement global concerne les variables déclarées avec le mot clé `var` mais à l’extérieur d'une fonction. Pour éviter ce problème une solution largement adoptée dans l’écosystème JavaScript est d'envelopper tout le contenu d'un fichier dans une fonction anonyme appelée immédiatement :

```javascript
(function(){
  // Code de votre fichier ici...
  // Ici la variable `x` est locale à ce fichier.
  // Sans l'utilisation de la fonction anonyme elle aurait été globale.
  var x = 5;
})();
```

La fonction enveloppe est appelée une seule fois immédiatement après avoir été définie, et le code qu'elle contient est exécuté normalement.

Bien, il est temps d'en revenir à Meteor. Meteor enveloppe automatiquement tous vos fichiers d'une fonction anonyme appelée immédiatement. Cela devrait vous éviter de modifier involontairement l'environnement global. Ainsi avec Meteor :

* Utilisez `var` pour déclarer une variable locale. À l'intérieur d'une fonction la variable sera locale à la fonction. À l'extérieur d'une fonction la variable sera locale au fichier.
* N'utilisez pas `var` pour déclarer une variable globale. Une telle variable sera accessible par tous les fichiers de votre application, ce qui est utile par exemple pour une collection.

```javascript
var a = 5; // Variable locale au fichier
var b = function() {
  var c = 5; // Variable locale à la fonction b;
};
x = 5; // Variable globale
```

En l'absence de mécanisme d'import/export de noms, l'utilisation de variables globales est le seul moyen de partager des variables entre plusieurs fichiers.

> Notez que Meteor oblige les paquets à déclarer dans un fichier descriptif les variables globales qu'ils exportent. Si un paquet utilise une variable globale sans l'exporter explicitement, cette variable sera globale seulement pour ce paquet, mais elle ne sera pas visible ailleurs (depuis d'autres paquets, ou depuis votre application). Pour cela Meteor effectue une analyse statique du code source des paquets. Nous y reviendrons dans le chapitre consacré à l'écriture de vos propres paquets.

Enfin, il arrive parfois que vous souhaitiez une libraire externe à votre application qui ne fonctionnera pas s'il est définie à l’intérieur d'une fonction anonyme auto-appelée (en particulier s'il elle utilise le mot clé `var` pour modifier l'environnement global, ce qui ne fonctionne pas avec Meteor). Pour corriger ce problème vous pouvez soit modifier manuellement le code source de cette libraire et supprimer le mot clé `var` devant la définition de variable globales. L'autre solution consiste à placer cette librairie dans un répertoire nommé `compatibility` pour indiquer à Meteor qu'il ne faut l’envelopper dans une fonction anonyme auto-appelée.

## Ordre des fichiers

L'écriture de votre application en utilisant un paradigme déclaratif devrait la rendre peu dépendante à l'ordre dans lequel les fichiers sont chargés.



* Les fichiers présents dans un sous répertoire sont chargés avant ceux du répertoire parent. Cela signifie que les fichiers les plus profonds dans l’arborescence sont chargés en premiers, et que les fichier à la racine de l'application sont chargés en dernier.
* À profondeur équivalente, à l'intérieur d'un même répertoire, les fichiers sont chargés par ordre alphabétique.

XXX `lib` avant le reste pour les libraires tierces

Par ailleurs si deux actions doivent êtres effectuées l'une après l'autre vous devriez les appeler depuis le même fichier, ce qui permet de s'assurer que l'ordre d’exécution sera respecté (au lieu de placer dans des fichiers différents et de vous reposer sur l'ordre alphabétique). Il existe une dernière exception au chargement des fichiers par ordre alphabétique dans un même répertoire : un fichier nommé `main.js`, ou plus généralement `main.*` sera chargé en dernier. Si deux fonctions globales `actionA` et `actionB` définies respectivement dans les fichiers `fileA.js` et `fileB.js` doivent être appelées dans cet ordre on créera un fichier `main.js` comme suit :

```javascript
actionA();
actionB();
```

Et au l'enregistrera dans le même répertoire que les deux fichiers précédents :

```
├── fileA.js
├── fileB.js
└── main.js
```

À l'heure actuelle il n'est pas possible de personnaliser ces règles. C'est toutefois une fonctionnalité demandée par la communauté et qui devrait arriver dans une prochaine version.

Par ailleurs si une portion de code nécessite que l'application soit chargée vous devriez l'envelopper dans une fonction passée en paramètre `Meteor.startup` :

```javascript
if (Meteor.isClient) {
  Meteor.startup(function() {

  });
}
```

## Proposition de conventions

En fait, il y a pas beaucoup de règles. Une fois que vous les maitriserez, la construction de l'application sera complètement automatisé.

Notez que les répertoires `client` et `server` peuvent être des sous répertoires, ce qui permet d'organiser son projet avec des modules de premier niveau, par exemple `/blog` et `/forum` à l'intérieur desquelles se trouvent les répertoire `client` et `server`.
