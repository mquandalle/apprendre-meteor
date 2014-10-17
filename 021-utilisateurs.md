  # Accueillons nos utilisateurs

Dans le « monde réel » nous ne voulons pas laisser tous le monde lire et modifier les données de l'application que ce fut le cas pour le leaderboard et le chat de la première partie. Notre application a besoin d'un système d'utilisateurs offrant les fonctionnalités classiques : inscription, connexion, oubli du mot de passe, connexion avec les « réseaux sociaux », etc. Cette gestion d'utilisateur est une tâche absolument récurrente des applications web, et on ose à peine imaginer son nombre d’implémentations et de ré-implémentations. Rassurez-vous, dans ce chapitre nous n'allons pas réinventez la roue une nouvelle fois. Meteor propose en effet un ensemble de paquets qui permettent de mettre en place un système d’utilisateurs d'une manière particulièrement simple pour le développeur, voyez plutôt.

## Un système d'utilisateurs complet en 60 secondes

Avant de démarrer il vous faut choisir l'application pour laquelle vous allez ajouter la gestion des utilisateurs, vous pouvez l'une des application de la partie précédente (leaderboard ou chat), ou bien utiliser une nouvelle application vierge. Une fois que l'application est choisie vous pouvez enclencher le chronomètre :

1. Ajoutez les paquets `accounts-password` et `accounts-ui` :

   ```bash
   $ meteor add accounts-password accounts-ui
   ```

2. Dans l'un des templates ajouter l'inclusion `{{> loginButtons}}`

Et voilà ! Les utilisateurs peuvent maintenant s’inscrire et se connecter à votre application avec un couple email/mot de passe.

> Vous pouvez changer l'alignement du menu de connexion avec l'attribut `align`, par exemple: `{{> loginButtons align="right"}}

[TODO] Lien vers Meteorpad

Naturellement en 60 secondes, nous n'avons rien *développé* de nouveau, nous nous sommes contenté d'*utiliser* du code existant. Cette utilisation fait sens pour un système comme le système d'utilisateur qui ne constitue probablement pas la valeur ajoutée de votre application. Par ailleurs utiliser le code fourni par meteor assure une meilleure qualité que si vous aviez (re)programmer l'ensemble du système. Des éléments comme le formulaire d'oubli de mot de passe sont souvent négligés et codés rapidement, et contiennent parfois des failles de sécurité (mots de passes envoyés par email, lien de réinitialisation du mot de passe réutilisable, etc).

L'utilisation du système d'utilisateurs fourni par meteor limite-t-il les possibilités de personnalisation que nous aurions eut en développant notre propre système ? Heureusement non, le système fourni par meteor est configurable à souhait, et si cela ne suffit pas, il est possible de l’étendre via des nouveaux paquets `accounts`.

## La galaxie des paquets `accounts`

Par convention, tous les paquets ayant trait à la gestion des utilisateurs sont préfixés par l'expression `accounts-`. On trouve ainsi des paquets officiels comme `accounts-password`, `accounts-facebook` ou `accounts-twitter`, on bien des paquets communautaires comme `joshowens:accounts-entry` ou `ian:accounts-ui-bootstrap-3 `.

Sans surprise ces paquets ajoutent des fonctionnalités au système utilisateur, vous pouvez par exemple essayer d'en ajouter un à l'application précédente :

```bash
$ meteor add accounts-twitter
```

[TODO] Lien vers Meteorpad

Ce paquet ajoute la possibilité de s'inscrire et de se connecter à votre application en utilisant son compte twitter, et ajoute un bouton à cet effet dans le menu `{{> loginButtons}}`.

Un clic dessus


Les paquets `accounts-facebook` et `accounts-google` fonctionnent de la même manière. Si vous souhaitez implémenter une authentification en utilisant une API d'un site tiers, vous devriez commencer par regarder sur atmosphere si celà n'a pas déjà été publié par quelqu'un d'autre. Sinon, vous pouvez créer votre propre paquet sur le modèle d'`accounts-twitter`, un chapitre de la partie « Notion avancées » est consacré à l'écriture de vos propres paquets.

## Fonctionnement

`accounts-base` fourni le minimum vital au système d'utilisateurs. Vous n'avez pas besoin de l'ajouter explicitement à votre application car c'est une dépendance des paquets plus avancés comme `accounts-password` utilisé à la section précédente.

En fait ce paquet définie simplement une collection dédiée par convention aux données des utilisateurs, cette collection est accessible via l'objet `Meteor.users`. Dans cette collection chaque document correspond à un utilisateur. Puisqu'il s'agit d'une collection tout à fait normale, vous pouvez utiliser toutes les méthodes que nous avons étudié dans le chapitre sur les collections, par exemple pour connaitre le nombre d'utilisateur inscrits :

```javascript
Meteor.users.find().count()
```

Bien que vous soyez absolument libre de définir les champs que vous voulez dans cette collection, les différent paquets `accounts-` respectent le schema suivant :

```json
{
  _id: "bbca5d6a-2156-41c4-89da-0329e8c99a4f",
  username: "cool_kid_13", // nom unique
  emails: [
    // chaque addresse email appartient à un utilisateur unique
    { address: "cool@example.com", verified: true },
    { address: "another@different.com", verified: false }
  ],
  createdAt: Wed Aug 21 2013 15:16:52 GMT-0700 (PDT),
  profile: {
    // Par défaut, le champ `profile` est accessible en écriture à l'utilisateur
    name: "Joe Schmoe"
  },
  services: {
    facebook: {
      id: "709050", // facebook id
      accessToken: "AAACCgdX7G2...AbV9AZDZD"
    },
    resume: {
      loginTokens: [
        { token: "97e8c205-c7e4-47c9-9bea-8e2ccc0694cd",
          when: 1349761684048 }
      ]
    }
  }
}
```

Par défaut, le champ `profile` est le seul accessible en écriture. Vous pouvez néanmoins personaliser ce comportement en utilisant les règles `allow` et `deny` par exemple:

```javascript
// On autorise l'écriture du champ `description`
Meteor.users.allow({
  update: function(userId, doc, fieldNames) {
    if (user._id !== userId)
      return false;

    if (fields.length !== 1 || fields[0] !== 'description')
      return false;

    return true;
  },

  fetch: ['_id']
});
```

> Notez qu'il est préférable de suivre les conventions de meteor. Dans le cas ci-dessus, le champ `decription` aurait plutôt sa place dans le `profile`.

En plus de la collection `Meteor.users`, le paquet `accounts-base` vous donne accès à un certain nombre de méthodes utiles. Par exemple `Meteor.userId()` retourne l'identifiant de l'utilisateur connecté, et `null` si l'utilisateur n'est pas connecté. De manière similaire `Meteor.user()` retourne le document correspondant à l'utilisateur connecté. Meteor oblige, ces deux méthodes sont des sources réactives :

```javascript
Deps.autorun(function() {
  if (Meteor.userId())
    alert("Hello!");
  else
    alert("Bye!");
});

(Ne faites pas ça.)

[TODO] Meteorpad

Côté client, vous avez également accès au helper `{{currentUser}}`, qui vous permet de tester si un utilisateur est connecté `{{#if currentUser}}` et qui permet également d'afficher une propriété de l'utilisateur, par exemple son nom d'utilisateur : `{{currentUser.username}}`.

[TODO] Intégrer au chat

[TODO] Utiliser alanning/roles

---

Si vous avez pratiqué d'autres frameworks web, tels que Symfony ou Rails, vous savez qu'il est souvent (inutilement) complexe de paramétrer correctement un système d'utilisateur, et que c'est encore pire quand on veut rajouter des fonctionnalités comme le login avec Facebook, Twitter, ou Google. Si Meteor propose un eintégration beaucoup plus simple, c'est grâce à son système d' « unipaquet » qui permet d'ajouter du code à la fois sur le client et sur le serveur qui fonctionne ensemble sans que le développeur n'est besoin d'écrire manuellement la colle pour relier les deux.

## Questions

