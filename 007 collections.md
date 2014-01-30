# Les collections

Après les *Templates* et les *Sessions*, les **Collections** sont le troisième objet élémentaire utilisé par l'application leaderboard. Jusqu'à présent nous nous sommes concentré sur le code exécutée du côté client, mais nous avons aussi besoin d'un serveur pour sauvegarder la liste des joueurs associée à leur score - sans quoi les modifications d'un utilisateur seraient perdues dès qu'il se déconnecte. Autrement dit nous avons besoin de *persister des données*, et c'est précisément le rôle des collections. 

## Persister des données depuis le serveur

Du côté serveur, une collection est simplement une API pour exécuter des requêtes en base de donnée. Par défaut Meteor propose d'utiliser la base de donnée MongoDB que nous présentons donc dans ce chapitre. Il est néanmoins possible d'utiliser d'autres systèmes de base de donnée mais leur intégration avec l'écosystème Meteor manque pour le moment de maturité.

> Le terme *collection* est issu du monde NoSQL (pour Not Only SQL) et correspond à la notion de *table* dans les bases de données relationnelles (MySQL, PosgreSQL, Microsoft Access, Oracle, etc.). Une collection permet de stocker des *documents*, notion analogue à celle de *ligne*.

```javascript
maCollection = new Meteor.Collection('collectionNameOnTheDatabase');
```

Le constructeur `Meteor.Collection` prend en premier paramètre le nom de la collection utilisé dans MongoDB. Si cette collection n'existe pas, une collection vide sera créée. Ce constructeur retourne un objet, nommé ici `maCollection`, possédant les méthodes pour interagir avec les documents de la collection - c'est à dire en insérer, en modifier ou en supprimer. On utilise en général une variable globale pour pouvoir accéder à cet objet - et donc aux données - depuis n'importe quel fichier de l'application.

> * avec JavaScript la déclaration d'une variable globale se fait en omettant le mot clé `var`
> * avec CoffeeScritpt on préfixe le nom de la variable par `@`. En fait `@var` est compilé en `this.var`. À l’extérieur des fonctions `this` est l'objet `window`, le même pour tous les fichiers. Attacher une variable à l'objet `window` permet donc de simuler une variable globale. Une fois cette variable déclarée, vous devez l'utiliser sans la préfixer par `@`.

Il est également possible d'utiliser les collections sans persister les données dans la base de donnée en utilisant le paramètre `null` lors de la création de la collection. Cela permet d'utiliser l'API des collections pour interagir avec des documents présents en mémoire. Ces documents seront donc perdus lors de l'arrêt du serveur.

```javascript
collectionLocale = new Meteor.Collection(null);
```

MongoDB ne nécessite pas de schéma prédéfini, vous n'avez donc pas besoin de définir des colonnes avec un nom et un type et vous pouvez insérer n'importe quel document JSON. Lors de l'insertion d'un document, MongoDB ajoute automatiquement un index nommé par défaut `_id`. La méthode `insert` retourne l'identifiant du document inséré.

```javascript
var documentId = maCollection.insert({
  message: "Hello World",
  version: 1.0
});
```

> Sous le capot, Meteor utilise ici la bibliothèque *Fibers* qui permet d'écrire dans un style synchrone du code qui s’exécute en fait de manière asynchrone. Un chapitre de la partie "Meteor avancé" est consacré à ce sujet.

Vous pouvez ensuite utiliser cet identifiant pour modifier le document :

```javascript
maCollection.update(
  { 
    _id: documentId
  }, 
  {
    message: "Hello New World",
    version: 1.5
  }
);
```

Le premier paramètre est le sélecteur et le second est le nouveau document. Il est également possible d'utiliser un modificateur :

```javascript
maCollection.update(
  { 
    _id: documentId
  }, 
  {
    $set: {
      version: 2
    }
  }
);
```

Dans cet exemple, l'utilisation du modificateur `$set` permet de garder le reste du document inchangé. L'API détaillée de MongoDB est décrite dans une annexe.

Par défaut si le sélecteur correspond à plusieurs documents, un seul sera modifié - pour des raisons de performance. Ce comportement peut être changé avec l'option `multi` indiquée en troisième paramètre.

```javascript
// Sélectionne tous les documents en version 2, et les met à jour en version 3
maCollection.update({version: 2}, {$set: {version: 3}, {multi: true});
```

Puisqu'il est courant de sélectionner un unique document à partir de son attribut `_id` il existe une forme raccourcie pour le sélecteur :

```javascript
maCollection.update(documentId, {$set: {version: 2.5});
```

Enfin la suppression d'un document se réalise avec la méthode `remove` qui prend comme paramètre un sélecteur MongoDB ou sa forme réduite :

```javascript
maCollection.remove(documentId);
```

> Le terme SQL pour supprimer un document est `DELETE`. Malheureusement il s'agit d'un mot réservé en JavaScript et il est ainsi interdit d’appeler une variable, un attribut d'objet ou une méthode `delete`. On a donc choisit un synonyme : `remove`. Il est probable que vous écriviez intuitivement `delete` au lieu de `remove` ce qui provoquera une erreur.

Tout comme la méthode `update`, `remove` dispose également d'une option booléenne `mutli`, fausse par défaut.

## En ligne de commande

[TODO] À détailler

La commande `meteor mongo` vous donne accès à un shell interactif pour interagir avec la base de données de votre application. Pour le moment il faut que l'application soit déjà lancée.

> `meteor mongo` fonctionne également avec les application déployées dans le nuage Meteor. Utilisez simplement `meteor mongo monappli.com`. Si vous avez défini un mot de passe il vous sera demandé.

## Minimongo : une base de donnée côté client !

Dans la plupart des applications web, un client JavaScript qui souhaite accéder à des données utilisera une méthode *Ajax* avec une fonction *callback* recevant sont résultat si tout c'est bien passé. Cette architecture implique l'écriture d'instructions très différentes selon que l'on accède aux données depuis le client (il faut écrire une requête AJAX) ou depuis le serveur (il faut écrire une requête SQL). Or l'utilisation de la même API sur le client et le serveur est un des principes fondateur de Meteor. Le framework propose ainsi d'utiliser **minimongo**, une base de donnée côté client - et implémenté en JavaScript - qui propose la même API que MongoDB sur le serveur. 

Ainsi on crée une base de donnée côté client de la même manière que sur le serveur :

```javascript
maCollection = new Meteor.Collection('collection');
```

Si le paramètre `collection` correspond à une collection existante dans la base de données sur le serveur, ses données seront automatiquement synchronisés avec la base de données sur le client. Ce mécanisme de synchronisation est basé sur le système de publication/souscription qui sera détaillé dans un prochain chapitre.

Cette simple déclaration vous donne donc accès à toutes vos données et aux méthodes `insert`, `update`, `delete`, `upsert`, etc. vous permettant d'interagir avec elles. Cette ligne de code étant identique à celle que nous devons écrire sur le serveur, il est commode de la déplacer à l’extérieur des blocs `Meteor.isClient` et `Meteor.isServer`, ce qui est utilisé dans l'application leaderboard.

Comme sur le serveur, il est possible d'utiliser une base de donnée "anonyme" sur le client avec l'option `null`. Les données seront locales à une instance de l'application et perdues lorsque l'utilisateur fermera l'onglet.

En revanche si vous voulez récupérer l'identifiant d'un document tout juste inséré, le code suivant ne fonctionnera pas sur le client :

```javascript
var documentId = maCollection.insert({content: 'Test'});
```

Cette instruction ne fonctionne comme attendu que sur le serveur. Sur le client vous devez utiliser un style asynchrone, c'est à dire indiquer en dernier paramètre à la méthode `insert` une fonction callback :

```javascript
maCollection.insert({content: 'Test'}, function(err, res) {
  var documentId = res;
});
```

La signature de fonction `(err, res)` est une convention largement utilisée dans l'environnement Node.js. `err` contient l'erreur ou `null` si l'opération s'est déroulée sans erreur. `res` contient l'identifiant du document nouvellement inséré. Il est également possible d'utiliser le style asynchrone sur le serveur.

|                   | Client  | Serveur |
|-------------------|:-------:|:-------:|
| Style asynchrone  | Oui     | Oui     |
| Style synchrone   | **Non** | Oui     |

Comme pour toutes les autres bibliothèques disponibles côté client, nous pouvons tester minimongo directement dans la console du navigateur. Dans l'application leaderboard vous pouvez par exemple exécuter la requête suivante :

```Javascript
> Players.findOne({name: "Nikola Tesla"});
Object {_id: "6c53bbe4-5013-4d87-ad4d-957230f12d60", name: "Nikola Tesla", score: 150}
```

Cette requête s’exécute sur la base de donnée locale, aucune communication n'est effectuée avec le serveur. Dans le cas ou l'utilisateur n'a pas accès à tous les documents (nous verrons comment ultérieurement), cette requête peut ne retourner aucun résultat sur le client alors qu'elle en retourne sur le serveur - ou sur un autre client.

### Ajoutez et supprimer des joueurs

Il est temps de compléter les deux fonctionnalités que nous avions commencées dans le chapitre "Templates" : l'ajout et la suppression de joueurs. Avec minimongo, c'est un jeu d'enfant.

Pour l'insertion on utilise la variable `playerName` précédemment définie, et on initialise un score de zéro points :

```javascript
Players.insert({name: playerName, score: 0});
```

Pour la suppression on utilise simplement la variable `this._id` comme sélecteur :

```javascript
Players.remove(this._id);
```

Et voilà ! Ces modifications sur la base de données locale sont automatiquement synchronisées avec le serveur (qui peut éventuellement les refuser comme nous le verrons plus loin) et qui les renvoie à tous les clients connectés, lesquels mettent à jour l'interface utilisateur de manière réactive. Instantanément !

## Les curseurs, encore une source réactive

Pas de réactivité sans source réactive. Si l'interface utilisateur est automatiquement mise à jour lorsqu'un nouveau joueur est ajouté, c'est parce que l'objet curseur, retourné par la méthode `.find()` est une source réactive. Lorsque qu'un document correspondant au curseur est inséré, modifié ou supprimé cela provoque une invalidation de contexte.

Il est donc tout à fait possible d'utiliser cette source réactive avec `Deps.autorun`. Ici nous utilisons par exemple la méthode `.count()` d'un curseur qui retourne le nombre de documents correspondants : 

```javascript
Deps.autorun(function () {
  console.log('nb_players: ' + Players.find().count())
});
```

Ce code générera une nouvelle sortie log dans la console à chaque fois que le nombre de documents est modifié. Un curseur est réactif par défaut, mais il est possible de désactiver ce comportement avec l'option booléenne `reactive` :

```javascript
Deps.autorun(function () {
  var min_score = Session.get('min-score');
  var cursor = Players.find({score: {$gt: min_score}}, {reactive: false});
  console.log(cursor.count() + 'players have a score greater than ' + min_score);
});
```

Ce contexte réactif sera automatiquement ré-exécuté si la variable de session est modifié, mais ne le sera pas si les documents de minimongo sont modifiés car nous avons désactiver la réactivité. 

### Exploiter les résultats d'un curseur

Avec l'API classique de MongoDB la méthode `find` retourne directement une liste des résultats - et non un curseur. Avec Meteor, vous devez utiliser la méthode `.fetch()` pour récupérer cette liste à partir du curseur :

```javascript
> Players.find().fetch();
[
  {_id: "6c53bbe4-5013-4d87-ad4d-957230f12d60", name: "Sarah Bitterlin", score: 475},
  {_id: "191aaefc-f582-495d-bb32-c531ee2db698", name: "Martin Stebler", score: 510},
  ...
]
```

Les curseurs possèdent également les méthodes `forEach` et `map` qui fonctionnent comme pour les listes classiques. Ces deux méthodes acceptent en unique paramètre une fonction qui sera exécutée à chaque itération :

```javascript
// Un exemple avec forEach
var totalScore = 0;
Players.find().forEach(function (doc) {
  // On ajoute le score de chaque joueur au compteur totalScore
  // Si le champ score d'un joueur n'est pas défini, on ajoute 0
  totalScore += doc.score || 0;
});
console.log('Total score : ' + totalScore);

// Un exemple avec map
var namesLength = Players.find().map(function (doc) {
  // On souhaite récupérer une liste contenant la longueur du nom
  // de chaque joueur
  return doc.name && doc.name.length;
});
```

Les méthodes `fetch`, `forEach` et `map` ne peuvent être appelées qu'une seule fois pour un curseur donné. La méthode `rewind` permet de réinitialiser le curseur :

```javascript
> allPlayersCursor = Players.find();

> allPlayersCursor.fetch();
[Object, Object, Object, Object, Object]

> allPlayersCursor.fetch(); // Sans réinitialiser le curseur
[]

> allPlayersCursor.rewind();

> allPlayersCursor.fetch();
[Object, Object, Object, Object, Object]
```

Si le curseur est réactif - c'est à dire si l'option `reactive` n'a pas été définie à `false` - les méthodes `count`, `fetch`, `forEach` et `map` conservent la réactivité.

### Observateurs

Il est possible de définir un observateur de curseur. Un observateur est un dictionnaire de fonctions qui sont appelées à chaque fois qu'un document correspondant au curseur est ajouté, modifié ou supprimé. Cela permet de déclencher une action sur tous les clients connectés lorsque des données sont modifiés.

Nous allons utiliser un observateur pour émettre une notification de navigateur à chaque fois qu'un joueur est supprimé. Cette popup de notification sera visible par tous les clients connectés, y compris ceux qui naviguent sur un autre onglet ou sur une autre application que la notre.

![Notification HTML5 émise par une application web](img/notification.png)

> Les notifications HTML5 ne fonctionnent que sur les versions modernes de Firefox, Chrome et Safari. Cette démonstration ne fonctionne donc pas sur Internet Explorer ou Opéra. La liste des navigateurs supportés est disponible sur le site [caniuse.com](http://caniuse.com/notifications).  
> Vous pouvez éventuellement remplacer le constructeur `Notification` par un simple `console.log`.

Un observateur se définit avec la méthode `observe` de l'objet curseur. Nous allons observer tout les documents et nous utilisons donc le curseur `.find()` sans critère de sélection.

```javascript
Players.find().observe({
  removed: function (player) {
    new Notification("BREAKING NEWS: " + player.name + " died.");
  }
})
```

Vous pouvez maintenant essayer cette fonctionnalité avec plusieurs clients connectés : supprimez un joueur depuis le premier client, les autres clients connectés recevront immédiatement une notification de la triste nouvelle.

La méthode `observe` retourne un manager qui permet éventuellement d'arrêter l'observation avec la méthode `stop`. Nous allons par exemple arrêter d'émettre des notifications si un joueur nommé "Meteor" est supprimé :

```javascript
var observerManager = Players.find().observe({
  removed: function (player) {
    if (player.name.toLowerCase() === "meteor") {
      new Notification("It's the end of Meteor, I stop to emit notifications.");
      observerManager.stop();
    } else {
      new Notification("BREAKING NEWS: " + player.name + " died.");
    }
  }
});
```

La méthode `observe` accepte en unique paramètre un dictionnaire de callbacks. Les fonctions suivantes sont disponibles :

* callbacks *sans* positionnement
    * `added(newDocument)`
    * `changed(newDocument, oldDocument)`
    * `removed(oldDocument)`
\
* callbacks *avec* positionnement
    * `addedAt(newDocument, atIndex, before)`
    * `changedAt(newDocument, oldDocument, atIndex)`
    * `removedAt(oldDocument, atIndex)`
    * `movedTo(document, fromIndex, toIndex, before)`

Les méthodes `added`, `changed` et `removed` sont plus efficaces que leur équivalent retournant également la position du document. Si vous n'avez pas besoin d'utiliser la position du document utilisez donc ces méthodes.

Les paramètres `oldDocument` et `newDocument` dans les signatures de ces fonctions contiennent les documents complets. Il peut être parfois pratique de ne récupérer que la différence entre l'ancien et le nouveau documents. Ceci est possible avec la méthode `observeChange` qui fonctionne de manière analogue à la méthode `observe` mais avec les callbacks suivants :

* callbacks *sans* positionnement
    * `added(id, fields)`
    * `changed(id, fields)`
    * `removed(id)`
\
* callbacks *avec* positionnement
    * `addedBefore(id, fields, before)`
    * `movedBefore(id, before)`

Le paramètre `fields` est un dictionnaire de tous les champs modifiés associés à leur nouvelle valeur. Une modification du champ `_id` est considéré comme la création d'un nouveau document et la suppression de l'ancien.

Comme tout à l'heure les callbacks retournant la position du document sont moins performant que les callbacks sans cette information. Le document courant doit être placé avant celui identifié par le champ `before`, ou à la fin si `before` vaut `null`.

En pratique on utilise les observateurs à chaque fois que l'on souhaite interfacer un objet non réactif avec des données réactives. L'exemple le plus courant est une carte interactive affichant des pointeurs modifiés en temps réel. Pour cela if faut interfacer les callback `added` et `removed` de l'observateur avec vos propres fonctions `addPin` et `removePin`.

## Sécurité: Gestion des droits d'écriture

### Le package `insecure`

Pour l'instant il suffit à un visiteur d'ouvrir la console de son navigateur pour pouvoir modifier le contenu de notre base de donnée. En production cela peut poser un petit problème que je vous propose de corriger tout de suite :

```sh
$ meteor remove insecure
```

Le package `insecure` permet d’accélérer le début du développement en autorisant à tous les clients l'ajout, la modification et la suppression de l'ensemble des données. Une fois le stade du prototypage dépassé, il est nécessaire de définir soit même les règles de validation de ces opérations.

Après avoir supprimé le package `insecure`, toutes les opérations sont interdites par défaut. Ainsi les fonctionnalités que nous avons implémenté dans leaderboard (ajout et suppression d'un joueur, incrément du score) ne fonctionnent plus, et il nous faut définir une à une les autorisations sur la collection `Players`.

### Les autorisations avec `allow`

Nous allons par exemple autoriser la suppression d'un joueur seulement si son score est au minimum de 100 points. Nous pourrions effectuer ce contrôle dans le callback, après avoir cliqué sur la croix et juste avant de supprimer le joueur de la base de données, mais un utilisateur mal intentionné pourrait toujours utiliser la console du navigateur et outrepasser cette vérification. Il est donc absolument nécessaire d'effectuer ce contrôle du coté serveur. Nous utilisons pour cela la méthode `allow` de notre collection :

```javascript
Players.allow({
  remove: function (userId, doc) {
    return doc.score > 100;
  }
);
```

La méthode `allow` reçoit en unique paramètre un dictionnaire contenant les callbacks pour les opérations `insert`, `update` et `remove`. L'opération sera autorisée si et seulement si le callback est défini et qu'il retourne `true` - ou une valeur évaluée comme vraie. Chaque callback reçoit en premier paramètre l'`userId` qui permet de retourner une autorisation dépendant des droits de chaque utilisateur. Nous étudierons le système d'utilisateur dans un prochain chapitre.

Ici on se contente de définir une règle simple : la suppression est autorisée si le score du joueur est supérieur à 100. Vous pouvez vérifier que cette règle fonctionne correctement dans le navigateur.

Définissons maintenant les autorisations pour l’insertion et la modification de joueurs :

```javascript
Players.allow({
  insert: function (userId, doc) {
    return false;
  },
  update: function (userId, doc, fieldsName, modifier) {
    return fieldsName.indexOf('name') === -1;
  },
  remove: function (userId, doc) {
    return doc.score > 100;
  }
});
```

La signature du callback `insert`, `(userId, doc)`, est identique à celle de `remove`, ici on choisit d'interdire l’insertion de tous les documents. En revanche le callback `update` reçoit des paramètres supplémentaires :

`doc`
  ~ contient le document tel qu'il est présent dans la base de donnée, c'est à dire avant la modification proposée

`fieldsName`
  ~ contient une liste des champs modifiés. Cette liste contient uniquement les champs de premier niveau. Ainsi si l'attribut `stats.age` est modifié seul le champ `stats` sera présent dans cette liste.

`modifier`
  ~ contient le modificateur MongoDB brut, par exemple `{$set: {'stats.age': 5}}`

Dans l'exemple précédent on interdit la modification du nom du joueur en vérifiant que le champ `name` n'est pas présent dans la liste des champs modifiés. Toutes les autres modifications sont autorisées.

Pour des modèles plus sophistiqués, la validation des données au sein d'une unique fonction peut devenir complexe. Il est possible de définir plusieurs règles `allow` pour une même opération, cette dernière sera alors autorisée si au moins un callback retourne `true`.

Par exemple nous pouvons ajouter la règle suivante pour l'insertion d'un joueur :

```javascript
Players.allow({
  insert: function (userId, doc) {
    var nameLength = doc.name && doc.name.length;h
    return doc.name && doc.name.length >= 5;
  }
});
```

La première règle `allow` définie précédemment retournant toujours `false`, seule cette deuxième règle régira l'insertion des documents.

> Si l'attribut `name` de l'objet `doc` n'existe pas, l'expression `doc.name.length` provoquera une erreur au moment de l’exécution car l'objet `undefined` ne possède pas d'attribut `length`. Il faut donc vérifier deux conditions dans cet ordre :
> 
> 1. Le joueur a un nom
> 2. La longueur de ce nom est supérieure ou égale à 5
>
> Cette double vérification s'écrit : 
> 
> ```javascript
> doc.name && doc.name.length >= 5
> ```
> 
> CoffeeScript propose l'opérateur `?.` pour éviter de ce répéter. L'expression JavaScript ci-dessus est ainsi équivalente à l'expression CoffeeScript suivante :
> 
> ```coffee
> doc.name?.length >= 5
> ```
>
> Notons enfin que les utilisateurs de CoffeeScript peuvent chaîner les inégalités :
> 
> ```coffee
> 20 > doc.name?.length >= 5
> ```

### Les interdictions avec `deny`

En plus de ces règles `allow` il est possible de définir des règles `deny` qui fonctionnent d'une manière similaire et permettent de s'assurer que certaines opérations sont interdites. Par exemple pour interdire la création d'un joueur au nom d'oiseau :

```javascript
Players.deny({
  insert: function (userId, doc) {
    var blackList = ['cigogne', 'pelican', 'faucon', 'coucou'];
    return blackList.indexOf(doc.name) !== -1;
  }
})
```

Il est également possible de définir plusieurs règles `deny` pour une même opération. Lorsqu'un client essaye d'écrire dans une collection, le serveur commence par vérifier toutes les règles `deny`. Si aucune d'elles ne retourne `true`, le serveur vérifie alors les règles `allow`. L'écriture est autorisée si et seulement si aucune règle `deny` ne retourne `true` et au moins une règle `allow` retourne `true`.

En général, on utilise le paramètre `userId` pour définir des règles en fonction des prérogatives d'un utilisateur. Ce système sera présenté ultérieurement et les règles définies ici pour l'application leaderboard ont pour seul intérêt d'illustrer le modèle de sécurité de validation des données.

### Modèle de sécurité

Toutes les autorisations/interdictions sont centralisées au niveau de la collection, ce qui est résolument une bonne chose. Néanmoins ce n'est peut-être pas un modèle que vous avez utilisé précédemment.

Il est en effet assez courant de récupérer les données d'un formulaire, puis de vérifier que ces données sont valides, puis si toutes les conditions sont vérifiées, de réaliser l'opération dans la base de donnée. Voici par exemple un code PHP qui suit ce modèle :

```php
if (strlen($_POST['player']) < 5) {
  $bdd->exec('
    INSERT INTO players (name, score)
    VALUES ('. mysql_real_escape_string($_POST['player']) .', 0)
  ');
}
```

Le problème c'est que l'instruction `mysql.insert` ne vérifie pas elle-même la condition `strlen($_POST['player']) < 5` avant de réaliser l'insertion du joueur. Autrement dit on suppose que l'instruction est exécutée avec des données qui ont été vérifiées précédemment. Si une autre partie de code de l'application réalise également cette insertion, il faudra valider les données à cet endroit là aussi. Ainsi il arrive parfois que des hackers arrivent à contourner les règles de sécurité sur des sites aussi populaires que facebook, par exemple en passant par l'interface mobile du site, dont les règles de validations peuvent différer de l'interface classique.

Définir toutes les conditions de validation au niveau de la collection avec les méthodes `Collection.allow` et `Collection.deny` permet de s'assurer qu'il sera impossible de les contourner. En effet les méthodes `insert`, `update` et `remove` vérifient systématiquement ces conditions avant d'effectuer les opérations, et il n'est donc pas nécessaire de vérifier les paramètres donnés à ces requêtes. Ce modèle de sécurité est particulièrement rendu nécessaire par l'utilisation de *minimongo* côté client qui permet à n'importe qui d'essayer d’exécuter n'importe quelle requête, et il ne faut donc pas se reposer sur le fait que les requêtes sont faites avec des données déjà validés.

### Restrictions sur les contextes non sécurisés

Le modèle de sécurité de Meteor est battit sur le principe de non confiance dans le code exécuté du côté client. En effet, un client peut tout à fait exécuter du code différent de celui que nous lui avons envoyé. On considère donc le code client comme un "contexte non sécurisé". Au contraire le code coté serveur est un "contexte sécurisé".

Pour des raisons de sécurité, Meteor impose les restrictions suivantes pour les requêtes `update` et `remove` dans un contexte non sécurisé :

* Les requêtes ne peuvent modifier qu'un seul document à la fois. L'option `multi` n'est pas disponible.
* Le seul sélecteur autorisé est l'identifiant du document, les sélecteurs MongoDB se basant sur d'autres champs que l'identifiant sont interdits
* L'opération `upsert` est interdite

```javascript
// Sélecteur interdit
> Players.update({name: "Nikola Tesla"}, {$set: {score: 100}});

// Sélecteur autorisé
> Players.update(playerId, {$set: {score: 100}});

// Le sélecteur fonctionne, mais il est déprécié.
// Il est conseillé d'utiliser la forme réduite (c'est à dire la 
// requête précédente)
> Players.update({_id: playerId}, {$set: {score: 100}});

// Opération upsert interdite
> Players.upsert({_id: playerId, name: "Nikola Tesla" , score: 100});
```

Notez que le package `insecure` supprime la notion de contexte non sécurisé, et supprime en conséquence les restrictions détaillées ci-dessus. Au cours du développement il peut donc être utile de rajouter temporairement ce package afin d'autoriser toutes les opérations de la base de données directement dans la console du navigateur.

> Le fonctionnement de la faille de sécurité ayant conduit à l'établissement de ces limitations est expliqué dans [ce fil](https://groups.google.com/forum/?hl=fr&fromgroups=#!searchin/meteor-talk/security/meteor-talk/qUa3clv4g5E/N7F8IaAX5AgJ) (en anglais). En un mot, la faille consiste à deviner des champs secrets (comme le mot de passe chiffré d'un utilisateur) via l'utilisation de sélecteurs bien choisis.

Ces restrictions peuvent être très contraignantes dans certains cas. Nous verrons dans le chapitre sur les *méthodes* comment créer un contexte sécurisé du côté client, ce qui nous permettra de les outrepasser.

--------------------------------------------

Implémenter une base de donnée du côté client est résolument un point fort en faveur de Meteor. Celle-ci s'intègre parfaitement avec les autres packages de l'écosystème pour proposer une synchronisation automatique ou encore des curseurs réactifs. Les collections implémentent également un modèle de sécurité robuste et cohérent avec le fonctionnement global du framework.

Après avoir étudié les Templates, les Sessions et les Collections, vous en savez maintenant assez pour construire l'application leaderboard. Cela peut vous sembler beaucoup de travail pour une application relativement simple, mais toutes ces notions seront réutilisées dans chaque application Meteor et vous ferrons gagner un temps précieux. Dès le prochain chapitre nous étudierons `parties` une autre application fournie avec Meteor. 

En attendant, libre à vous d'ajouter encore quelques fonctionnalités au `leaderboard`. Je mets à votre disposition un dépôt sur github contenant l'historique de l'ensemble des modifications effectuées lors de ces sept premiers chapitres.

## Questions

