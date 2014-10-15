# Pourquoi Meteor ?

Avant d'écrire nos premières lignes de code quelques petites présentations s'imposent. Après un historique rapide du développement web et introduit les arguments en faveur de l'utilisation d'un framework en général et de Meteor en particulier.

## Petit historique du développement web

<blockquote>
« Septembre 1993 restera dans l'histoire du net comme le mois de septembre éternel. »<br>
— Dave Fischer
</blockquote>

Au début des années 1990 Tim Berners-Lee conçoit une nouvelle forme de navigation décentralisée basée sur un protocole - le HTTP - et sur un logiciel - le navigateur - capable de lire des documents structurés contenants des liens hypertextes vers d'autres documents potentiellement situés sur d'autres serveurs. Le World Wide Web est rapidement adopté par quelques universités américaines qui l'utilisent pour s'échanger des documents écris en HTML, le langage de balisage inventé pour écrire ces documents structurés.

Naturellement, des développeurs ont l'idée de créer des documents dynamiques, c'est à dire générés automatiquement. Rien n'empêche en effet un serveur de retourner une réponse différente à chaque requête qu'il reçoit. Le premier langage de programmation utilisé pour générer des pages dynamiques fut le Perl, mais c'est avec l'arrivée du PHP en 1995 que le développement d'applications web prend véritablement son envol. De très nombreuses applications web sont toujours écrites en PHP dont les géants Facebook et Wikipedia.

Fortes de près de vingt années d'utilisation, les technologies de développement web côté serveur ont aujourd'hui largement gagné en maturité. Il est maintenant possible d'utiliser à peu près tous les langages de programmation (Python, Ruby, PHP, Java, Go, C, etc.) pour générer automatiquement une réponse à une requête HTTP. Des fonctions courantes de ces applications web sont par exemple l'interrogation d'une base de donnée, l'écriture d'un fichier sur le serveur, ou encore l'authentification d'un utilisateur par un couple pseudo/mot de passe.

Parallèlement à ces évolutions côté serveur et grâce aux efforts conjugués des grands noms du Web (Mozilla, Google, Adobe, etc.) le JavaScript, au départ simple language de script utilisé pour des usages assez peu intensifs, est progressivement devenu un langage de programmation complet et de plus en plus performant. Son utilisation est en effet nécessaire pour écrire des applications s’exécutant directement dans le navigateur web côté client, un modèle qui permit par exemple le succès des applications les plus populaires de Google parmi lesquelles Maps, GMail ou encore Google Docs.

Par rapport aux applications côté serveur, les applications côté client rendent possible une meilleure expérience utilisateur : elles sont plus rapides, ne rechargent pas les pages (permettant ainsi de faire des transitions animées) et peuvent être réactives (une modification d'un utilisateur est immédiatement propagée à tous les clients connectés). Elles sont cependant difficile à construire à partir de rien car elles nécessitent de faire communiquer ensemble le serveur et le client sur tout le long de la chaîne de composants : requête Ajax, websockets, mise à jour du DOM en JavaScript, mise à jour de la base de données du serveur, etc.

## Et c'est ici qu'arrive Meteor

Meteor permet d'écrire des application web modernes d'une manière beaucoup plus simple, ceci justement grâce au contrôle de l'ensemble de l'environnement client et serveur. Ainsi avec Meteor par exemple, si un nouveau commentaire est ajouté à un article de mon blog, il apparaîtra instantanément chez tous les clients connectés sans que je n'ai besoin d'écrire la moindre ligne de code spécifique pour cette fonctionnalité ! Tous les composants de la chaîne de communication des données (la connexion à la base de donnée, la communication client/serveur et la mise à jour de l'interface) doivent donc fonctionner de manière « réactive » en temps réel.

![Le logo de Meteor](img/meteor-logo.png)

Meteor s'appuie sur Node.js qui permet d’exécuter du code JavaScript sur le serveur. Cela signifie que nous allons utiliser le même langage de programmation sur le client et sur le serveur, un confort indéniable pour le programmeur. Meteor pousse même l'idée un peu plus loin en unifiant les API utilisées sur le client et sur le serveur, on parle de JavaScript isomorphique, un terme un peu pédant pour désigner ce concept qui vous semblera bientôt naturel. Prenez par exemple la méthode `Meteor.startup` qui permet d’exécuter du code au démarrage du client ou du serveur. Du côté serveur la fonction est exécutée dès que le processus Node.js est lancé, et du côté client elle l'est dès que le DOM est prêt à être modifié. Il s'agit donc d'une méthode disponible sur les deux environnement pour le même usage, exécuter du code au démarrage, mais implémentée de manière différente sur le client et sur le serveur.

![Le logo de Node.js, la plateforme sur laquelle s'appuie Meteor](img/nodejs-logo.png)

C'est dans cet esprit que Meteor propose également une base de donnée complète accessible du côté client. À l'usage vous verrez qu'il est très commode de pouvoir faire des requêtes directement dans le navigateur avec la même API que sur le serveur. Il s'agit de l'API de MongoDB qui est la base de donnée utilisée par défaut avec Meteor. Un développeur utilisera donc les mêmes méthodes sur deux environnement différents pour exprimer la même idée (« stocker des données ») bien que l'implémentation de ces méthodes différe largement en fonction des contraintes imposées à l'environnement (il n'est pas possible d'éxecuter du code natif dans un navigateur web par exemple).

> Il possible d'utiliser d'autres bases de données sur le serveur mais pour le moment leur intégration avec l'écosystème Meteor manque de maturité et nous utiliserons donc exclusivement MongoDB dans ce cours.

Le JavaScript isomorphique devient encore plus intéressant quant on considère que le serveur web (l'application node.js) et le client (l'application qui s'éxecute dans le navigateur web) ne sont que des cas particulier d'une notion plus générale de *plateforme*. Autrement dit, Meteor va nous permettre de développer nos applications pour d'autres environnements : Android, iOS, extension de navigateur, outil en ligne de code, etc. Un chapitre de ce cours est consacré au développement d'application mobiles avec Apache Cordova.

Le code source de Meteor est *libre* et publié sous la très permissive licence MIT. Il est hébergé sur [github](https://github.com/meteor/meteor) et accepte les « Pull Request » (propositions de modifications) de la part des membres de la communauté. Le développement du framework est néanmoins largement assuré par la société **Meteor Development Group** dite **MDG** qui [a levé 11,2 millions de dollars en 2012](http://www.meteor.com/blog/2012/07/25/meteors-new-112-million-development-budget) assurant la pérennité du projet à moyen terme et permettant à une équipe de développeurs talentueux de s'y consacrer à plein temps.

> *Faut-il connaitre Node.js pour utiliser Meteor ?*<br>
> Non. Node.js expose des API relativement bas niveau. En général le code que nous écrirons du côté serveur interagira avec des bibliothèques de plus haut niveau fournies par Meteor. Voyez cela comme une pile de couches, Meteor crée une couche d'abstraction qui s'appuie sur la couche Node.js. La maîtrise de cette dernière n'est donc pas directement nécessaire. Néanmoins si vous avez de l’expérience avec Node.js elle vous sera évidemment utile pour des usages plus avancés.

## Pourquoi utiliser un framework ?

L'une des questions récurrente lorsqu'un développeur souhaite se former au développement web concerne l’intérêt d'utiliser un framework plutôt que d'écrire du code « brut ».

Traduit littéralement de l'anglais, un framework est un « cadre de travail ». Il fournit un certains nombre de briques élémentaires (pour Meteor les *paquets*) conçues pour fonctionner de manière complémentaire, et utilisables par un développeur pour créer des applications avancées sans s'attarder sur les éléments génériques (persister des données dans une base de donnée, associer une URL à une action, gérer un système d'inscription et de connexion pour des utilisateurs, etc.) ; c'est à dire se concentrer exclusivement sur la *logique* et l'*interface* propre de son application.

Comme pour tous les autres frameworks, apprendre à utiliser Meteor nécessite d'y consacrer un peu de temps d'étude et de pratique, mais le jeu en vaut la chandelle ! Le temps d’apprentissage sera largement récupéré par la suite, et vous surprendrez à développez en quelques heures des applications qui vous aurez autrefois requis des jours de travail.

Enfin outre ce gain d'organisation et de productivité, Meteor permet de rendre le développement web moins fastidieux, et donc plus intéressant — voire *amusant*.

## Et pourquoi pas avec d'autres outils ?

L'autre question qui tracase le développeur avant de s'embarquer dans une nouvelle technologie est « Pourquoi XXX et pas YYY ? ». En effet ce ne sont pas les outils qui manquent pour développer des applications web, de nouveaux apparaisant (puis disparaissant) presque tous les jours. Pourquoi donc Meteor et pas Rails, Django, jQuery, AngularJS, React, Grunt, Famous,  {inserer_votre_technologie_ici} ?

Il faut déjà situer Meteor par rapport à toutes ces technologies éparses. Meteor est un framework web « full stack » c'est à dire agissant partout : sur le serveur, le navigateur, l'application mobile, les communications inter-plateformes, etc. Il n'y a en fait qu'assez peu de projet relevant la même mission. [Derby](http://derbyjs.com) et [Sails](http://derbyjs.com) semblent toutefois poursuivre les mêmes objectifs. Je vous laisse consulter le site de ces projets pour glaner plus d'informations à leur propos, mais à l'heure ou j'écris ces lignes leur niveau d'avancement et leur niveau d'engagement communautaire est en général moindre que celui de Meteor.

Et puis il y a les autres technologies, celles que j'ai citées plus haut. Il faut comprendre ces librairies ne remplissent pas le même rôle que Meteor. En fait, il est possible d'intégrer la plupart de ces technologies dans l'ecosystème Meteor. Cette intégration se réalise via le système de paquets, qui permet d'ajouter ou de remplacer des briques logicielles de votre application. S'il est effectivement possible d'intégrer ces technologies externes avec Meteor, je vous recommande vivement d'attendre d'avoir une compréhension fine de Meteor avant d'en changer les composants fournis par défaut.

## Ressources utiles

Parallèlement à la lecture de cet ouvrage, si vous souhaitez poser une question ou approfondir un sujet en particulier, vous pouvez bénéficier de l'écosystème prolifique de Meteor :


* La **[documentation](http://docs.meteor.com)** est la ressource indispensable. Accessible, complète, à jour, elle contient la réponse aux problèmes les plus courants. Notez qu'elle est accessible même hors connexion car elle est intégralement stockée dans le cache du navigateur. Nous verrons comment créer des applications hors ligne dans un chapitre de ce cours.
* L'étiquette `meteor` sur [StackOverflow](http://stackoverflow.com) constitue une excellente base de questions/réponses. Si votre question n'a pas déjà été posée par quelqu'un d'autre vous pouvez vous inscrire et la publier, vous recevrez généralement une réponse dans les heures qui suivent.
* La [liste de discussion générale](http://groups.google.com/group/meteor-talk) située ici est l'endroit idéal pour les remarques, questions techniques, offres d'emplois ou n'importe quelle discussion en rapport avec Meteor. Il existe aussi [une liste réservée aux discussions concernant le développement du framework](http://groups.google.com/group/meteor-core) en lui même.
* Le site [crater.io](http://crater.io) est un aggrégateur d'actualité pour l'écosystème Meteor inspiré d' « Hacker-News » ou de « Reddit ». Vous y trouverez les liens les plus populaires du moment : nouveaux paquets de la communauté, exemples d'application, articles de blog...
* L'application [MeteorPad](http://meteorpad.com) qui permet de partager des applications Meteor et de les executer dans une machine virtuelle depuis le navigateur. Certains exemples de cet ouvrage seront accompagné d'un lien vers MeteorPad, ce qui permettra de les modifier sans rien télécharger sur votre PC.

La communauté Meteor accueille chaleureusement les débutants, et une fois la lecture de cette ouvrage terminé vous êtes évidemment invité à perpétuer ce bon accueil ;-)

Place au code !
