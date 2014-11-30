# Je publie, tu souscris

Lors du chapitre sur les collections nous avons défini des règles de sécurité pour restreindre l'écriture sur la base de donnée d'une application Meteor. Mais pour le moment tous les utilisateurs conservent un accès en lecture à toutes les données de l'application. Remédions-y dans ce chapitre.

## Concept

Une application web peut traiter des volumes de données gigantesques, parmi lesquelles se trouvent notamment des informations privées. La base de données Minimongo exécutée côté client ne peut donc pas être une simple copie conforme de la base de donnée réelle côté serveur. Au contraire elle doit contenir un sous-ensemble de documents, à la fois pour des raisons de performances et de sécurité des données.

Meteor utilise le modèle du publication/souscription pour normaliser les échanges de données entre le client et le serveur. Bien qu'il soit rarement utilisé de manière aussi explicite, ce modèle est parfaitement adapté aux applications web et devrait vous sembler assez intuitif. Dans ce modèle, le serveur *publie* des jeux de données auxquels le client peut souscrire, c'est à dire qu'il demande de recevoir les données. Un client avancé pourra donc gérer ses souscriptions pour précharger des données, garder des informations dans le cache minimongo, ou même souscrire à l'intégralité des données publiées par le serveur à des fins d'archivage.

[TODO] Image souscription publication

## Sécurité : Le package `autopublish`

Les applications que nous avons rencontré dans la première partie de ce cours ne déclaraient aucune publication ni aucune souscription. Ce absense était rendue possible par l'utilisation du paquet `autopublish` ajouté par défaut lors de la création d'une nouvelle application Meteor, qui publie automatiquement toutes les données de la base de données au client. Ce paquet est ajouté pour accélérer le prototypage, aussi une fois que vous êtes prêt à définir vos propres règles de publications, vous pouvez le retirer :

```sh
$ meteor remove autopublish
```

Tout comme le package `insecure`, le package `autopublish` ne devrait pas être utilisé dans une application en production.

## Publications

Imaginons une collection contenant les articles de mon blog. Certains de ces articles sont publiés et peuvent donc être consultés par tout le monde, d'autres sont en brouillon et je ne souhaite pas que mes visiteurs puissent y accéder. Je vais donc écrire la publication suivante côté serveur :

```javascript
Meteor.publish('posts', function() {
	return Posts.find({draft: false});
});
```

Le premier paramètre de la méthode `Meteor.publish` est le nom de la publication, le second paramètre est la fonction qui retourne les documents à publier. Ici on retourne les articles dont l'attribut `draft` vaut `false`.

> Il est possible de retourner n'importe quel type de document dans une fonction de publication, ceci le cas le plus commun consiste à retourner un curseur sur une collection. Des détails d'implémentation à ce propos seront présentés dans le chapitre « DDP » de la partie « Notions avancées ».

Les publications gèrent toutes les règles de sécurité relative à la publication des données. Ainsi si je n'ajoute aucune autre publication à mon application, il sera impossible pour un client d’accéder à des documents dont l'attribut `draft` vaudrait autre chose que `false`.

Les données étant envoyées au client par le serveur, il est tout à fait possible pour ce dernier de conditionner leur envoi selon le client. Pour se faire on utilisera le système d'utilisateurs présenté dans un chapitre précédent.

```
Meteor.publish('draftPosts', function() {
	if (this.userId === '') {
		return Posts.find({draft: true});
	}	else {
		// One fonction de publication doit forcément retourner un résultat
		return [];
	}
});
```

XXX faire avec meteor role is admin

XXX discuter des implications en terme de performance, ce qui n'est pas grave car cela concerne uniquement une publication destinée aux administrateurs.

Les fonctions de publications retournent un curseur réactif. Ainsi si un nouveau document correspondant au curseur est inséré dans la base de données côté serveur, il sera automatiquement envoyé à tous les client souscrivant à cette publication.

## Souscriptions

Au fur et à mesure du temps mon blog va s'enrichir d'articles et de commentaires. Aussi un utilisateur aura le droit d'accéder à une quantité de donnée toujours plus conséquente, mais ce n'est pas une raison pour tout lui envoyer automatiquement. De même, sans être inscrit sur un forum j'ai le droit de lire des centaines de messages, mais je dois pour cela le demander au serveur. C'est le principe d'une souscription :

```javascript
if (Meteor.isClient) {
	Meteor.subscribe('posts');
}
```

La méthode `Meteor.subscribe` prend en premier paramètre le nom de la publication définie côté serveur. Elle retourne un objet qui permet d'arrêter la souscription, les données seront alors supprimées de la base de donnée minimongo et le serveur n'enverra plus de mises à jour :

```javascript
if (Meteor.isClient) {
	var subscriptionHandler = Meteor.subscribe('posts');
	setTimeOut(function() {
		// On arrête la souscription après 10 secondes.
		subscriptionHandler.stop();
	}, 5000)
}
```

Il est également possible de transmettre des paramètres à la fonction de publication par exemple pour publier un article en particulier :

```javascript
if (Meteor.isServer) {
	Meteor.publish('post', function (postId) {
		return Posts.find({_id: postId, draft: false});
	});
}

if (Meteor.isClient) {
	Meteor.subscribe('post', 4);
}
```

Ici l'utilisateur demande l'article avec l'identifiant `4`. En général on utilise cette fonctionnalité avec une variable de session qui stocke de manière réactive l'article courant, celui que le visiteur consulte en se moment :

if (Meteor.isClient) {
	Tracker.autorun(function() {
		Meteor.subscribe('post', Session.get('current-post'));
	});
}

La souscription est située dans un contexte réactif `Tracker.autorun`. La modification de la variable de session (par exemple si l'utilisateur consulte l'article suivant) entraîne une ré-exécution du code et donc l'initialisation d'une nouvelle souscription. Mais mieux encore, Meteor va automatiquement stopper la souscription précédente (avec la méthode `.stop()` vue précédemment), de sorte que ce simple code est suffisant pour récupérer automatiquement les données dont on a besoin et oublier celles dont on n'a plus l'utilité.

## Présentation de `todos`

Tout comme l'application `leaderboard`, `todos` est l'une des applications exemple fournies par défaut avec meteor. Cette application est plus complète que le `leaderboard` et nous servira de modèle d'application pour le « monde réel ». La procédure de création est toujours la même :

```sh
$ meteor create --example todos
$ cd todos
$ meteor
```

XXX capture d'écran de l'application todos par défaut

L'application permet de créer et de gérer des listes de tâches. Par défaut trois listes sont instianciées et l'on choisir la liste courante en cliquant sur son nom dans le menu de gauche. Dans le volet de droite un clic sur s'importe quel texte permet de l'éditer. Les fonctions classiques d'ajout et de suppression d'élements fonctionnent comme attendu. Ces trois listes sont des listes « publiques » que tout le monde peut voir et modifier. Il est aussi possible de créer des listes « privées » en se connectant avec un compte utilisateur.

XXX Capture d'écran du formulaire d'inscription

Vous remarquez que le formulaire qui est utilisé dans cette application n'est pas le widget `accounts-ui` que nous avons vu précédemment mais un formulaire ré-implementé pour les besoins ergonomiques de l'application. La ré-implémentation concerne uniquement le formulaire de saise, le système d'utilisateur utilisé est bien celui que nous avons étudié précédemment. On retrouve parmi les autres conventions suivies par le code source de cette application la séparation du code client et serveur dans deux répertoires distincts, le placement des ressources dans le répertoire `public`, l'utilisation d'un fichier distinct pour définir chaque template, ou encore l'utilisation du même nom de fichier pour les fichiers `html` et `js` correspondants.

Comme pour l'application leaderboard, nous allons ajouter des nouvelles fonctionnalités à `todos` au cours des prochains chapitres.

### Partage des listes privées

Les listes privées ont un « propriétaire », *owner* en anglais, qui est le seul utilisateur qui peut y accéder. Nous allons ajouter la possibilité de « partager » les listes privées, c'est à dire les rendre accessible en lecture et en écriture à d'autres utilisateurs que son propriétaire.

L'interface utilisateur sera implémentée dans le prochain chapitre consacré aux « Méthodes », ici on se contente d'utiliser le système de publication-souscription pour donner accès à la liste aux amis de son propriétaire. Pour se faire, il vous faudra utiliser un outil pour modifier la base de donnée de votre application (par exemple l'application visuelle robomongo ou l'utilitaire en ligne de commande `meteor mongo`, tout deux présentés dans l'excursus consacré aux outils de développement).

XXX code pour créer deux utilisateurs userA-userA@localhost-passA, userB-passB

On ajoute dans le modèle un champ `friends` qui contiendra une liste des identifiants des utilisateurs aillant accès à cette liste.

```javascript
Meteor.publish('friendsList', function() {
  if (this.userId) {
    return Lists.find({friends: this.userId});
  } else {
    this.ready();
  }
});
```

## DDP



## Mergebox

---

Conclusion à propos du protocole de la séparation des données avec le protocole DDP
@TODO en fait toute les applications fonctionnent sur un modèle de publication souscription, mais ce modèle n'est en général pas explicité ou défini comme tel. Ce fait est favorable à Meteor. → En conclusion ?
