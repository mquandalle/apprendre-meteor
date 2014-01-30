# Je publie, tu souscries

Lors du chapitre sur les Collections nous avons défini des règles de sécurité pour restreindre l'écriture sur la base de donnée d'une application Meteor. Mais pour le moment tous les utilisateurs conservent un accès en lecture à toutes les données de l'application. Remédions-y dans ce chapitre !

## Concept

Une application web peut traiter des volumes de données gigantesques, parmi lesquelles se trouvent notamment des informations privées. La base de données Minimongo exécutée côté client ne peut donc pas être une simple copie conforme de la base de donnée réelle côté serveur. Au contraire elle doit contenir un sous-ensemble de documents, à la fois pour des raisons de performances et de sécurité des données.

Meteor utilise le modèle du publication/souscription pour normaliser les échanges de données entre le client et le serveur, ce qui permet au développeur de restreindre l'accès à un sous ensemble de documents.

## Sécurité : Le package `autopublish`

L'application leaderboard ne contient aucune publication ni aucune souscription. En fait le package `autopublish` ajouté par défaut lors de la création d'une nouvelle application Meteor, publie automatiquement toutes les données de la base de données au client. Une fois que vous êtes prêt à définir vos propres règles de publications, vous pouvez le retirer :

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

Le premier paramètre de la méthode `Meteor.publish` est le nom de la publication, le second paramètre est la fonction de publication. Cette règle rend strictement impossible l'accès à des documents dont l'attribut `draft` vaudrait autre chose que `false`. Les publications gèrent toutes les règles de sécurité relative à la publication des données.

Il est possible de retourner un résultat différent en fonction de conditions, par exemple pour autoriser la lecture de tous les articles à l’administrateur du blog.

Les fonctions de publications retournent un curseur réactif. Ainsi si un nouveau document correspondant au curseur est inséré dans la base de données côté serveur, il sera automatiquement envoyé à tous les client souscrivant à cette publication.

## Souscriptions

Au fur et à mesure du temps mon blog va s'enrichir d' articles et de commentaires. Un utilisateur aura donc le droit d'accéder à une quantité de donnée conséquente, mais ce n'est pas une raison pour tout lui envoyer automatiquement. De la même manière, même si je ne suis pas inscrit sur un forum j'ai le droit de lire des centaines de messages, mais je dois pour cela le demander au serveur. C'est le principe d'une souscription :

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
	Deps.autorun(function() {
		Meteor.subscribe('post', Session.get('current-post'));
	});
}

La souscription est située dans un contexte réactif `Deps.autorun`. La modification de la variable de session (par exemple si l'utilisateur consulte l'article suivant) entraîne une ré-exécution du code et donc l'initialisation d'une nouvelle souscription. Mais mieux encore, Meteor va automatiquement stopper la souscription précédente (avec la méthode `.stop()` vue précédemment), de sorte que ce simple code est suffisant pour récupérer automatiquement les données dont on a besoin et oublier celles dont ont a plus l'utilité.

## Présentation de `todos`



```sh
$ meteor create --example parties
```

## Sécurité : Le package `autopublish`



## Publications et souscriptions avancées

Conclusion à propos du protocole DDP détaillé dans un chapitre avancé
