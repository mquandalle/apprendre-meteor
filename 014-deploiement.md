# Excursus : Déploiement

Ce cours est parsemé de quelques chapitres excursus dont la lecture n'est pas requise pour comprendre le reste du cours. Celui-ci concerne le déploiement d'une application Meteor. Si vous voulez améliorer votre leaderboard avant de le montrer au monde entier, vous pouvez passer ce chapitre et y revenir plus tard.

Ce chapitre présente deux possibilités pour la mise en production d'une application: le nuage Meteor et votre propre serveur dédié.

## Déploiement dans le nuage Meteor

*Meteor Developpement Group* dite *MDG* est la société principale contributrice du projet Meteor. Elle met à disposition un hébergement dans le nuage, pour le moment gratuit. Y déployer son application est d'une simplicité enfantine :

```bash
$ meteor deploy monsousdomaine.meteor.com
```

Une fois l'opération terminée, rendez-vous sur <http://monsousdomaine.meteor.com> et voilà ! Votre application est maintenant publique et le monde entier peut y accéder.

> Vous pouvez également utiliser votre propre nom de domaine. Il faut pour cela définir un enregistrement DNS `CNAME` redirigeant vers `origin.meteor.com`. Vous pourrez ensuite déployer avec la commande précédente :
>
> ```bash
> $ meteor deploy www.mondomaine.com
> ```

Pour mettre à jour votre application, il vous suffit de développer les modifications en local, puis d’exécuter à nouveau la commande `meteor deploy`. Cela signifie que tout le monde peut écraser votre application. Pour vous réserver ce privilège vous pouvez définir un mot de passe qui vous sera alors demandé à chaque nouveau déploiement. Utilisez l'option `--password` ou sa forme raccourcie `-P` :

```bash
$ meteor deploy -P monsousdomaine.meteor.com
```

Vous pouvez accéder aux logs du serveur (le texte affiché *via* la méthode `console.log`) avec la commande `meteor logs www.mondomaine.com`. Si vous avez défini un mot de passe, il vous sera alors demandé.

Cette solution d’hébergement est gratuite pour le moment, mais MDG devrait proposer une offre commerciale dans les prochains mois.

## Paramètres

`Meteor.settings`
Variables d'environnement
`Meteor.release`

### Reproductibilité

Repetable builds

Meteor promises repeatable builds for packages, as well as apps -- if, say, you built your package on a machine, then checked the code into a repository and checked it out elsewhere, you should get the same result. For an app, we ensure consistency across builds by storing the dependency versions used to build the app in the .meteor/versions file. For a package, we do something similar with the versions.json file. Once you build your meteor package (by running tests, adding it to an app, etc), you will find a versions.json file in the package directory. That file is part of the source. Check it into version control to ensure repeatable builds across machines.

## Déploiement sur votre propre serveur

### Bundling

Minification du js et css. Compression des images.

[TODO] `meteor bundle`

### Galaxy

[TODO] Galaxy ?

[TODO] Stack personnalisé ? (installer node, npm, mongodb, fibers, forever ; déploiement avec git (git-receive))
