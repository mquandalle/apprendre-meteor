# Sessions et réactivité

## L'objet `Session`

Le terme *session* peut être source de confusion -- en particulier chez les développeurs web. Ce que Meteor appelle "session" n'est pas une variable persistante stockée côté serveur et récupérée à l'aide d'un identifiant de session transmis via un *cookie*. Il vous faut donc mettre de côté vos intuitions concernant le fonctionnement des variables de sessions Meteor, mais fort heureusement leur fonctionnement plutôt simple permet d'éviter une confusion avec la notion de session dans d'autres environnements de développement.

L'object `Session` de Meteor fournit un stockage :

1. exclusivement du côté client
2. par clé-valeur 
3. en mémoire
4. global

### Comment utiliser les sessions ?

On définit un couple clé-valeur avec la méthode `set` :

```javascript
Session.set('key', value);
``` 

où `'key'` est une chaîne de caractères et `value` n'importe quel objet JSON (y compris des objets simples comme un nombre ou une chaîne de caractères).

Pour récupérer la valeur associée à une clé, on utilise la méthode `get` :

```javascript
Session.get('key');
``` 

L'objet `Session` étant accessible depuis n'importe où sur le client, c'est un stockage qui est utile pour définir en un même endroit toutes les variables relatives à l'état courant de l'application (par exemple la position et le niveau de zoom sur une carte interactive).

En plus des méthodes `get` et `set`, l'objet Session dispose d'une méthode `setDefault` qui permet de définir une valeur par défaut si la variable de session n'a pas encore été définie avec un `Session.set`. Cette valeur par défaut correspond en général à l'état initial de l'application.

L'API de ce système de stockage est donc relativement simple :

```javascript
> Session.setDefault('a', 0);
> Session.get('a');
0

> Session.set('a', 3);
> Session.get('a');
3
```

Ce qui différencie une variable de session d'une simple variable JavaScript est la notion de réactivité. Si vous utilisez la valeur d'une variable de session à l'intérieur d'un template helper, celui-ci sera automatiquement mis à jour à chaque fois que la valeur attachée à la clé sera modifiée. 

Dans l'application leaderboard une variable de session est utilisée pour gérer la sélection d'un joueur. Cliquez sur un joueur, puis dans la console du navigateur récupérez l'identifiant du joueur sélectionné avec `id = Session.get('selected_player')`. Cliquez à présent sur un autre joueur, puis toujours dans la console faites un `Session.set('selected_player', id)`. L'interface est alors instantanément mise à jour, le joueur possédant l'identifiant que nous avions récupéré est à nouveau sélectionné.

### Application : sélectionner plusieurs joueurs à la fois

Nous allons modifier l'application leaderboard pour permettre à l'utilisateur de sélectionner plusieurs joueurs à la fois. On souhaite qu'un clic sur un joueur inverse sa sélectivité : on le sélectionne s'il ne l'est pas et on le désélectionne s'il l'est déjà.

Pour implémenter cette fonctionnalité on remplace la variable de session `selected_player` par une variable `selected_players` qui contiendra une liste d'identifiants des joueurs sélectionnés. Par défaut aucun joueur n'est sélectionné et on définit donc une liste vide. Les sessions n’existant que du côté client, ce code doit être placé à l'intérieur de la condition `Meteor.isClient` :

```javascript
Session.setDefault('selected_players', []);
```

Il nous faut maintenant modifier les deux helpers et les deux événements utilisant notre ancienne variable de session.

On commence par modifier la condition déterminant si un joueur est sélectionné. Pour l'instant on vérifie simplement si la variable de session `selected_player` est égale à l'identifiant du joueur. Remplaçons ce code pour vérifier si l'identifiant du joueur est l'un des éléments du tableau des joueurs sélectionnés avec la méthode `indexOf` de notre liste :

```javascript
Template.player.selected = function () {
  return Session.get("selected_players").indexOf(this._id) !== -1 ? "selected" : '';
};
```

On modifie ensuite l'helper `selected_name` qui affiche le nom du joueur sélectionné. S'il n'y a qu'un seul joueur sélectionné on garde la logique actuelle, s'il y a un plus d'un joueur sélectionné on va écrire "n joueurs sélectionnés", et si il n'y a aucun joueur sélectionné on retourne `false` à l'attention de la condition `{{#if selected_name}}`. Cela nous donne l'helper suivant :

```javascript
Template.leaderboard.selected_name = function () {
  var selectedPlayers = Session.get("selected_players");
  switch (selectedPlayers.length) {
    case 0:
      return false;
    
    case 1:
      var player = Players.findOne(selectedPlayers[0]);
      return player && player.name;
    
    default:
      return selectedPlayers.length + " selected players";
  }
};
```

Reste maintenant à mettre à jour nos deux éventements. Premièrement au clic sur l'`input.inc` on incrémente le score de chaque joueur dont l'identifiant est dans la liste `selected_players` à l'aide d'une boucle `for`:

```javascript
Template.leaderboard.events({
  'click input.inc': function () {
    var selectedPlayers = Session.get("selected_players");
    for (var i = 0; i < selectedPlayers.length; i++)
      Players.update(selectedPlayers[i], {$inc: {score: scoreIncrement}});
  }
});
```

Enfin lors d'un clic sur un joueur on inverse l'état du joueur:

* s'il est dans le tableau, on le supprime
* s'il n'y est pas, on l'ajoute

```javascript
Template.player.events({
  'click': function () {
    var selectedPlayers = Session.get("selected_players");
    var index = selectedPlayers.indexOf(this._id);
    if (index !== -1) // On désélectionne le joueur
      selectedPlayers.splice(index, 1);
    else // On le sélectionne
      selectedPlayers.push(this._id);
    Session.set("selected_players", selectedPlayers);
  }
});
```

Sauvegardez les modifications. Vous pouvez maintenant donner des points à plusieurs joueurs à la fois !

> Le *Hot Code Push* conserve les variables de sessions. Ce n'est pas le cas si on ouvre un nouvel onglet ou si on recharge la page. Ces actions équivalent à une nouvelle session et donc à une réinitialisation des variables de sessions.

## La réactivité

### Introduction au paradigme de programmation réactive

Comme vous l'avez peut-être remarqué, le fichier `leaderboard.js` ne contient aucun code dédié à la manipulation du DOM. Sans Meteor, vous auriez normalement du gérer l'ajout de la classe `selected` avec ce type de code :

```javascript
document.getElementById('lineId').className = "selected";
```

Et vous auriez également du écrire quelque chose d'analogue pour supprimer la classe. Certaines librairies populaires comme jQuery ou MooTools permettent de faire cette modification de manière plus élégante et plus consistante entre les navigateurs, mais le principe reste le même. Cette manière d'écrire le code correspond au paradigme de *programmation impérative*, la classe `selected` est ajoutée au moment où je le demande, puis l'instruction est "oubliée", et je dois demander une nouvelle action si je veux supprimer la classe.

Avec Meteor au contraire on se contente de *déclarer* une condition de sélectivité et le DOM est automatiquement modifié en fonction de cette condition. La paradigme à l’œuvre ici s'appelle la **programmation réactive**. L'utilisation de la programmation réactive est un élément majeur dans la sensation de "magie" que ressentent les débutants face à une interface mise à jour toute seule ; comprendre ce paradigme permet donc de démystifier cet aspect du framework.

> Notez que Meteor et jQuery/MooTools ne sont pas du tout incompatibles entre eux. Vous pouvez tout à fait utiliser ces libraires côté client, soit via un plugin (barre d'onglet, accordéon, etc.), soit en manipulant directement le DOM. Ils ne sont opposés ici que pour illustrer les paradigmes *impératif* et *réactif*.

Un tableur (comme Microsoft Excel ou OpenOffice Calc) est probablement l'exemple d'application réactive le plus connu. Imaginons que nous ayons une cellule `A1` contenant un nombre, et une cellule `A2` contenant la formule `=A1+1`. Ainsi si `A1` vaut `5`, `A2` vaudra 6. Si on change la valeur de `A1`, la valeur de `A2` est automatiquement recalculée et le nouveau résultat y est affiché. En revanche si on modifie la cellule `A3`, alors il est clairement inutile de recalculer `A2` car sa valeur dépend uniquement de `A1`. Pour construire une application réactive, il nous faut donc un système permettant :

1. De ré-exécuter un calcul plusieurs fois
2. De définir *quand* on doit ré-exécuter ce calcul

### `Deps`, le chef d'orchestre de la réactivité

Meteor dispose d'une libraire nommée `Deps` qui répond à ces deux besoins. Cette libraire n'est utilisable que du coté client. C'est elle qui est utilisée sous le capot par le système de templates pour mettre à jour l'interface à chaque fois qu'une variable de session est modifiée. Il est tout à fait possible de l'utiliser directement.

La méthode `autorun` est probablement celle que vous utiliserez le plus. Elle vous permet d’exécuter une fonction une première fois immédiatement, puis à nouveau à chaque fois que l'une de ses dépendances est modifiée.

Vous pouvez essayez le code qui suit dans une nouvelle application vierge : `meteor create reactivity && cd reactivity && meteor` :

```javascript
if (Meteor.isClient) {
  Session.setDefault('a', 10);

  Deps.autorun(function () {
    var b = Session.get('a') + 1;
    console.log('b = ' + b);
  });
}
```

La fonction fournie en unique paramètre à la méthode `autorun` est exécutée une première fois et produit le log `b = 11` dans la console immédiatement. Modifions la valeur de la variable de session dans la console avec un `Session.set('a', 100);`, la fonction est alors automatiquement ré-exécutée et produit un nouveau log `b = 101`. En revanche si vous modifiez une autre variable de session, par exemple `Session.set('c', 'hello')`, la fonction ne sera pas ré-exécutée.

> *Un peu de vocabulaire*  
> La fonction que nous avons donnée en paramètre à la méthode `autorun` s'appelle un *contexte réactif*. Ce contexte dispose d'un certain nombre de *dépendances*. Lorsqu'une ou plusieurs de ses dépendances sont modifiées, on dit que le contexte est *invalidé*.

Au chapitre précédent nous avons vu qu'il est possible d'englober les templates helpers dans des fonctions. Cela permet en fait de créer un contexte réactif. Comme pour `Deps.autorun` la fonction est exécutée une première fois et son résultat est affiché dans l'interface, puis ré-exécutée à chaque invalidation de contexte.

Vous aurez peut-être parfois besoin d'utiliser la variable booléenne `Deps.active` qui vaut vrai uniquement à l'intérieur d'un contexte réactif :

```javascript
Deps.autorun(function() {
  console.log('Deps.active inside Deps.autorun: ' + Deps.active);
});

console.log('Deps.active outside reactive context: ' + Deps.active);
```

Seules les *sources réactives* peuvent invalider un contexte réactif. Ainsi le code suivant ne produit qu'un seul log dans la console avec la valeur `a = 1`, mais la fonction n'est pas ré-exécutée lorsque l'on assigne une nouvelle valeur à la variable `a`.

```javascript
var a = 1;
Deps.autorun(function() {
  console.log('a = ' + a);
});
a = 4;
```

### `Meteor.status()`

Les variables de sessions ne sont pas les seules sources réactives. Un certain nombre d'objets fournis par Meteor côté client sont également capable d'invalider un contexte. C'est le cas par exemple de `Meteor.status()` qui retourne un objet contenant l'état de la connexion entre le client et le serveur. Cet objet contient les champs suivant :

`connected`
  ~ Valeur booléenne qui indique si le client est actuellement connecté au serveur.

`status`
  ~ Une description de l'état de la connexion. Les valeurs possibles sont :
  * "connected" la connexion fonctionne
  * "connecting" déconnecté mais essaye d'initialiser une nouvelle connexion
  * "failed" définitivement déconnecté (par exemple si le client et le serveur fonctionnent avec des versions incompatibles)
  * "waiting" déconnecté, en attente d'un nouvel essai
  * "offline" le client s'est déconnecté

`retryCount`
  ~ Le nombre de fois qu'un client a essayé de se reconnecter depuis que la connexion a été perdue. Vaut 0 si le client est connecté.

`retryTime`
  ~ Une estimation de la date de la prochaine tentative de connexion.

`reason`
  ~ Si `status` vaut "failed", une description de l'erreur.

La réactivité de cet objet vous permet par exemple d'effectuer une action à chaque fois qu'un client se connecte ou se déconnecte du serveur :

```javascript
Deps.autorun(function() {
  if (Meteor.status().connected)
    console.log('Hello server');
  else
    console.log('Bye server');
});
```

Lancer le serveur, le log "Hello server" apparaît dans la console. Arrêtez le, la fonction est ré-exécutée et produit le log "Bye Server".

--------------------

Dans ce chapitre nous avons introduit le paradigme de *programmation réactive*, ainsi que deux sources réactives : `Session` et `Meteor.status`. Nous verrons lors du chapitre sur les collections que les *curseurs* sont une autre source réactive importante. Sous le capot, tous ces objets utilisent la même bibliothèque `Deps`. De ce fait, il est tout à fait possible de créer vos propres sources réactives en utilisant les méthodes de cette bibliothèque. Un chapitre de la partie "Meteor avancé" est dédié à l’implémentation de la réactivité dans un objet JavaScript.

## Questions

1. Quel est l’intérêt de stocker une valeur dans une variable de session plutôt que dans une variable simple ?
    - la valeur est conservée lorsque l'utilisateur recharge la page
    - la valeur est stockée dans un cookie
    - la valeur est réactive
\
2. Comment s'appelle la fonction donnée en paramètre à la méthode `Deps.autorun` ?
    - une source réactive
    - un contexte réactif
    - une dépendance
\
3. Qu'indique la variable `Deps.active` ?
    - Elle indique si l'objet courant est réactif
    - Elle indique si la réactivité est activé pour l'application
    - Elle indique si le contexte courant est réactif
    - Elle indique que `Deps` fonctionne
