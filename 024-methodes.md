# Les méthodes


Le client et le serveur doivent communiquer ensemble.

Communication entre le client et le serveur. Connexion persistante.

## Remote Procedure Call

En première approche les méthodes sont des fonctions que le client peut appeler et qui sont executées sur le serveur. Dans les applications client-serveur ce type de protocole est en général connu sous le sigle de RPC pour *Remote Procedure Call*. Dans le cas particulier d'une application web, le développeur dispose de plusieurs implémentations possible d'un protocole RPC. L'une d'elle est connue sous le sigle Ajax pour *Asynchronous JavaScript and XML* et consiste à initier des nouvelles requêtes HTTP depuis le client de manière asynchorne et sans recharger la page. Meteor propose une autre implémentation qui n'est pas directement basée sur le HTTP, mais qui utilise DDP, le protocole d'échange des données que nous avons présenté dans le chapitre précédent.

Malgré la profusion de sigles que contient cette explication, l'utilisation des méthodes est particulièrement simple.

Côté serveur on déclare une méthode avec la méthode `Meteor.methods` qui accepte en unique paramètre un dictionnaire du nom de chaque méthode associé à la fonction à executer :

```javascript
Meteor.methods({
  somme: function (a, b) {
    return a + b;
  }
});
```

Côté client on utilise `Meteor.call` et on indique d'une part le nom de la méthode à appeler et d'autre part la liste des arguments de la méthodes.

```javascript
Meteor.call("somme", 1, 1);
```

L'execution de la méthode est asynchrone côté client, ce qui veut dire que le client n'est pas bloqué à attendre la réponse du serveur. Pour exploiter le résultat d'une méthode, on utilise donc une fonction de retour :

```javascript
Meteor.call("Somme", 1, 1, function(err, res) {
  if (err)
    console.error('Erreur fatale : ', err);
  else
    console.log('1 et 1 font', res);
});
```

Meteor.Error pour retourner l'erreur au client

XXX exemple: division par zero

"divide-by-zero"


Comme nous l'avons vu au chapitre précédent, le canal de communication DDP supporte l'authentification des utilisateurs. Aussi à l'intérieur d'une méthode on peut accéder à l'identifiant de l'utilisateur courant avec `this.userId`.

Application: partager une todo liste par mail

if this.userId === null

listOwner = Lists.findOne(listId).owner;
if this.userId !== listOwner


## Compensation de latence

Attaquons-nous au principe numéro 4 de Meteor :

> *Compensation de latence*. Côté client, utiliser préchargement et simulation pour faire comme la connexion à la base de données était sans aucune latence.

Ce principe se traduit concrétement par la possibilité de définir des méthodes également du côté client. Meteor oblige, l'API utilisée est exactement la même que du côté serveur.

XXX possibilité d'appeler une méthodes depuis le serveur
XXX que se passe t-il quand on appelle une méthode à l'intérieur d'une autre méthode


## Contexte sécurisé

Notion de contexte sécurisé → pas de limitations sur les opérations, pas de vérification de règles allow/deny.


## Methodes versus opérations sur les collections

Dans quels cas préferer l'utilisation des méthodes au lieu d'un appel direct à une opération insert/update/remove ?
* requête compliquées
* performance (par exemple pour un agrégat, on ne va pas envoyer toutes les données au client simplement pour un count())
* timestamp


