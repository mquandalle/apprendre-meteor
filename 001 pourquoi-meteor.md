# Pourquoi développer avec Meteor ?

Avant d'écrire nos premières lignes de code quelques petites présentations s'imposent. Cette introduction présente un historique rapide du développement web et introduit les arguments en faveur de l'utilisation d'un framework en général et de Meteor en particulier.

## Petit historique du développement web

> *« Septembre 1993 restera dans l'histoire du net comme le mois de septembre éternel. »* -- Dave Fischer

Au début des années 1990 Tim Berners-Lee conçoit une nouvelle forme de navigation décentralisée basée sur un logiciel, le navigateur, capable de lire des documents structurés contenants des liens hypertextes vers d'autres documents potentiellement situés sur d'autres serveurs. Le World Wide Web est rapidement adopté par quelques universités américaines qui l'utilisent pour s'échanger des documents écris en HTML, le langage de balisage inventé pour écrire ces documents structurés.

Des développeurs ont ensuite l'idée de créer des documents dynamiques, c'est à dire générés automatiquement. Un serveur peut en effet retourner une réponse différente à chaque requête qu'il reçoit. Le premier langage de programmation utilisé pour le développement web fut le Perl, mais c'est avec l'arrivée du PHP développé par Rasmus Lerdorf en 1995 que le développement d'applications web prend véritablement son envol. De très nombreuses applications web sont toujours écrites en PHP dont les géants Facebook et Wikipedia.

Fortes de près de vingt années d'utilisation, les technologies de développement web côté serveur ont aujourd'hui largement gagné en maturité. Il est maintenant possible d'utiliser à peu près tous les langages de programmation (Python, Ruby, PHP, Java, Go, C, etc.) pour générer automatiquement une réponse à une requête HTTP. Des fonctions courantes de ces applications web sont par exemple l'interrogation d'une base de donnée, l'écriture d'un fichier sur le serveur ou l'authentification d'un utilisateur *via* un couple pseudo/mot de passe.

Parallèlement à ces évolutions côté serveur et grâce aux efforts conjugués des grands noms du Web (Netscape, Mozilla, Google, etc.) le JavaScript est progressivement devenu un langage de programmation complet et de plus en plus performant pour écrire des applications s’exécutant directement dans le navigateur web d'un client. Son utilisation a par exemple permis le succès des applications les plus populaires de Google parmi lesquelles Maps, GMail ou encore Google Docs.

Les applications côté client permettent une meilleure expérience utilisateur : elles sont plus rapides, ne rechargent pas les pages (permettant ainsi de faire des transitions animées) et peuvent être réactive (une modification d'un utilisateur est immédiatement propagée à tous les clients connectés). Elles sont cependant difficile à construire car elles nécessitent de faire communiquer ensemble tout une chaîne de composants : requête Ajax, websockets, mise à jour du DOM en JavaScript, etc.

## Et c'est ici qu'arrive Meteor

Meteor permet d'écrire des application web avancés d'une manière beaucoup plus simple. L'une des fonctionnalités caractéristique de Meteor est que les données sont en "temps réel" par défaut. Ainsi si un nouveau commentaire est ajouté à un article de mon blog, il apparaîtra instantanément chez tous les clients connectés sans que je n'ai besoin d'écrire la moindre ligne de code spécifique pour cette fonctionnalité ! Tous les composants de la chaîne de communication des données (la connexion à la base de donnée, la communication client/serveur et la mise à jour de l'interface) doivent donc fonctionner de manière "réactive" en temps réel.

![Le logo de Meteor, sans fioritures](img/logo.png)

Meteor s'appuie sur Node.js qui permet d’exécuter du code JavaScript sur le serveur. Autrement dit nous allons utiliser le même langage de programmation sur le client et sur le serveur ce qui est un confort indéniable pour le programmeur. Meteor pousse même l'idée un peu plus loin en unifiant les API utilisées sur le client et sur le serveur. Par exemple la méthode `Meteor.startup` permet d’exécuter du code au démarrage du client ou du serveur. Du côté serveur la fonction est exécutée dès que le processus Node.js est lancé, et du côté client elle l'est dès que le DOM est prêt à être modifié. Il s'agit donc d'une méthode disponible sur les deux environnement pour le même usage, exécuter du code au démarrage, mais implémentée de manière différente sur le client et sur le serveur.

![Le logo de Node.js, la plateforme sur laquelle s'appuie Meteor](img/nodejs-logo.png)

Dans cet esprit Meteor propose également une base de donnée accessible du côté client. À l'usage vous verrez qu'il est très commode de pouvoir faire des requêtes directement dans le navigateur avec la même API que sur le serveur. Il s'agit de l'API de MongoDB qui est la base de donnée utilisée par défaut avec Meteor. Il est théoriquement possible d'utiliser d'autres bases de données sur le serveur mais pour le moment leur intégration avec l'écosystème Meteor manque de maturité et nous utiliserons donc exclusivement MongoDB dans ce cours.

Le code source de Meteor est *libre* et publié sous la licence très permissive MIT. Il est hébergé sur [github](https://github.com/meteor/meteor) et accepte les "Pull Request" (propositions de modifications) de la part des membres de la communauté. Le développement du framework est néanmoins largement assuré par la société **Meteor Development Group** qui a levé 11,2 millions de dollars en 2012 assurant la pérennité du projet à moyen terme ^[<http://www.meteor.com/blog/2012/07/25/meteors-new-112-million-development-budget>] et permettant à une équipe de développeurs de s'y consacrer à plein temps.

[TODO] Évoquer les offres commerciales d’hébergement d'application quand elles seront dévoilées

> *Faut-il connaitre Node.js pour utiliser Meteor ?*  
> Non. Node.js propose des API relativement bas niveau. En général le code que nous écrirons du côté serveur interagira avec des bibliothèques de plus haut niveau fournies par Meteor. Néanmoins si vous avez de l’expérience avec Node.js elle vous sera évidemment utile pour des usages plus avancés.

## Pourquoi utiliser un framework ?

L'une des questions récurrente lorsqu'un développeur souhaite se former au développement web concerne l’intérêt d'utiliser un framework plutôt que d'écrire du code "brut".

Traduit littéralement de l'anglais, un framework est un "cadre de travail". Il fournit un certains nombre de briques élémentaires (pour Meteor les *packages*) conçues pour fonctionner de manière complémentaire, et utilisables par un développeur pour créer des applications avancées sans s'attarder sur les éléments génériques (persister des données dans une base de donnée, associer une URL à une action, gérer un système d'inscription et de connexion pour des utilisateurs, etc.) ; c'est à dire se concentrer exclusivement sur la *logique métier*.

Ce gain de productivité est particulièrement important avec Meteor. En effet le framework s'appuie sur Node.js qui est relativement difficile à prendre en main, en particulier parce que les méthodes de communication avec le réseau qu'il fournit sont de bas niveau (ce qui n'est pas le cas avec PHP par exemple).

Comme pour tous les autres frameworks, apprendre à utiliser Meteor nécessite d'y consacrer un peu de temps d'étude et de pratique, mais le jeu en vaut la chandelle ! Le temps d’apprentissage sera largement récupéré par la suite, et vous surprendrez à développez en quelques heures des applications qui vous aurez autrefois requis des jours de travail.

Enfin outre ce gain d'organisation et de productivité, Meteor permet de rendre le développement web moins fastidieux, et donc plus intéressant -- voire *amusant*.

## Ressources utiles

En parallèle de la lecture de cet ouvrage, si vous souhaitez poser une question ou approfondir un sujet en particulier, vous pouvez bénéficier de l'écosystème prolifique de Meteor :

[TODO] Présentation sous forme de tableau avec vignette des sites

* La documentation est située à l'adresse suivante <http://docs.meteor.com/>. Elle est simple, complète, à jour et contient la réponse à la plupart des problèmes rencontrés. Notez que la documentation est accessible même hors connexion car elle est intégralement stockée dans le cache du navigateur. Nous verrons comment créer des applications hors ligne dans un chapitre de ce cours.
* Le tag `meteor` sur <http://stackoverflow.com/> constitue une excellente base de questions/réponse. Si votre question n'a pas déjà été posée par quelqu'un d'autre vous pouvez vous inscrire et la publier, vous recevrez généralement une réponse dans les heures qui suivent.
* La liste de discussion générale située ici <http://groups.google.com/group/meteor-talk> est l'endroit idéal pour les remarques, questions techniques, offres d'emplois ou n'importe quelle discussion en rapport avec Meteor. Il existe aussi une liste réservée aux discussion concernant le développement du framework <http://groups.google.com/group/meteor-core>.
* Le site <http://crater.io/> est un "Hacker-News-like" pour l'écosystème Meteor; vous y trouverez les liens les plus populaires du moment : nouvelles versions du framework, exemples d'application, articles de blog...
* Le site <http://meteorpedia.com> est un wiki développé en Meteor est dédié à ce framework contenant de bon articles sur des points particuliers du développement ou de la mise en production.

La communauté Meteor accueille chaleureusement les débutants, et une fois la lecture de cette ouvrage terminé vous êtes évidemment invité à perpétuer ce bon accueil ;-)

Place au code !
