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

L'utilisant d'un paradigme déclaratif devrait rendre votre application peu dépendante à l'ordre dans lequel les fichiers sont chargés. Néanmoins il est difficile d'échapper à un certain nombre de situations où cet ordre entre en jeu. Meteor utilise des règles assez rigides pour déterminer dans quel ordre il charge les fichiers. Les deux principales règles sont :

* Les fichiers présents dans un sous-répertoire sont chargés avant ceux du répertoire parent. Cela signifie que les fichiers les plus profonds dans l’arborescence sont chargés en premiers, et que les fichier à la racine de l'application sont chargés en dernier
* À profondeur équivalente, à l'intérieur d'un même répertoire, les fichiers sont chargés par ordre alphabétique.

À l'heure actuelle il n'est pas possible de personnaliser ces règles. Il s'agit toutefois d'une fonctionnalité demandée par la communauté qui devrait arriver dans une prochaine version.

Il existe une exception bien utile pour le répertoire `lib` dont les fichiers sont chargés avant les autres. Il pourra contenir des librairies dont votre application dépend et qui doivent être chargées avant le reste de l'application, par exemple pour un plugin jquery côté client :

```
├── client
│   ├── fileusingdatetimepicker.js
│   ├── lib
│   │   └── jquery.datetimepicker.js
│   └── view.html
└── server
```

Ici le fichier `fileusingdatetimepicker.js` pourra utiliser les fonctions définies dans le plugin `jquery.datetimepicker.js`.

Par ailleurs si deux actions doivent êtres effectuées l'une après l'autre vous devriez les appeler depuis le même fichier, ce qui permet de s'assurer que l'ordre d’exécution sera respecté (au lieu de placer dans des fichiers différents et de vous reposer sur l'ordre alphabétique). Il existe une unique exception au chargement des fichiers par ordre alphabétique dans un même répertoire : un fichier nommé `main.js`, ou plus généralement `main.*` sera chargé en dernier. Si deux fonctions globales `actionA` et `actionB` définies respectivement dans les fichiers `fileA.js` et `fileB.js` doivent être appelées dans cet ordre on créera un fichier `main.js` comme suit :

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

## En pratique

Bien que le règles présentées ci-dessus puissent vous sembler nombreuses et contraignantes, il n'en ait rien, et à l'usage découvrirez une très grande flexibilité quant à l'organisation de vos applications. D'où la nécessité de mettre en place des conventions plus restrictives pour s'assurer un certain ordre.

On dénombre aux moins deux écoles en la matière. La première consiste à imposer une organisation dès le début, et à s'y tenir tout au long du développement. C'est l'approche retenue par la plupart des frameworks web tels que Symfony, Rails, ou encore Django. La contrepartie à l'assurance d'organisation que cette méthode garantie, c'est la difficulté à itérer des expérimentations rapidement, car ajouter et retirer des fonctionnalités nécessite systématiquement la modification de nombreux fichiers bien séparés et au rôle bien définis. Cette difficulté à itérer rapidement est en général amoindrie par l'utilisation d'outil en ligne de commande pour générer automatiquement du code, outils dit de *scalfolding*. L'autre approche consiste à ne se préoccuper de l'organisation du code qu'après expérimentation. Cela permet de créer des nouvelles fonctionnalités quitte à placer tout le code, client comme serveur, dans un unique fichier, et de ne réorganiser le code qu'une fois sûr du découpage à adopter.

Les développeurs Meteor pourront utiliser les deux approches. La seconde approche se traduira par l'utilisation d'un fichier unique `nomDeLaNouvelleFonction.js` à la racine du projet qui permettra de faire des expérimentations facilement. La première sera utile dès que l'on aura une idée précise de l'organisation à adopter. Les développeurs chevronnés n'auront pas systématiquement besoin d'en passer par une phase d'experimentation, et pourront développer et organiser le code en parallèle.

Les règles décrient dans les sections précédentes tendent naturellement à favoriser l'utilisation des répertoires particuliers comme des répertoires parents placés à la racine de l'application :

```
├── client
│   ├── fonctionA.css
│   ├── fonctionA.html
│   ├── fonctionA.js
│   ├── fonctionB.css
│   ├── fonctionB.html
│   └── fonctionB.js
├── lib
│   ├── collections.js
│   ├── methods.js
│   └── routes.js
├── private
├── public
├── server
│   ├── fonctionA.js
│   └── authorisations.js
└── tests
```

La première convention respectée dans cet exemple est de donner un même nom aux fichiers qui fonctionnent ensemble, ainsi on placera les template helpers et les évènements des templates définis dans le fichier `fonctionA.html` dans un fichier `fonctionA.js`. C'est une convention qu'il est recommandé de suivre car elle facilite grandement la recherche du code JavaScript et CSS associé à une vue.

Certains développeurs poussent l'idée plus loin et recommandent de ne définir qu'un seul template par fichier `html`. Cette convention est par exemple respectée par l'application `todos` que nous étudirons dès le prochain chapitre. Aussi on definira le template `<head>` dans un fichier `head.html` et le template `<body>` dans un fichier `body.html`. Cette convention ne semble par faire l'objet d'un consensus absolu, certains developpeurs considerant qu'il est préférable de définir plusieurs templates liés dans un même fichier, à vous de voir donc si vous souhaitez également suivre cette convention strictement, ou si cela fait sens de regrouper plusieurs templates dans le même fichier. Quoi qu'il en soit vous ne devirez pas définir plus de trois ou quatre templtes au sein d'un même fichier.

Votre application s'organisera probablement autour de plusieurs fonctionnalités telles qu'un blog ou un forum. Une manière de bien organiser la séparation de ces différentes fonctionnalités est de les définir dans des répertoires distincts, en s'appuyant sur le fait que les répertoires `client` et `server` peuvent être des sous répertoires :

```

```

Enfin, notez que si vous développez une fonctionnalité assez générique pour être réutilisée dans plusieurs application (un système de commentaires, un système d'avatar...) il convient de la déplacer dans un paquet indépendant assurant ainsi un cloisonement des espaces de noms. Un chapitre de la partie « Notions avancées » est consacrée à la création d'un paquet.

---

