# Apprendre Meteor (alpha-2)

> **Attention :** La rédaction de ce cours n'est pas terminée.

Ce cours en français va vous permettre de découvrir le framework
[Meteor](https://www.meteor.com/). Vous pouvez accéder à la version PDF
[ici](https://github.com/mquandalle/apprendre-meteor/releases).

Ce cours est publié sous la license
[Creative Commons BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.fr).

## Pré-requis

HTML, CSS, JavaScript

## Contribuer

Les contributions sont les bienvenues via les *Pull Requests* sur Github.
Vous pouvez également me contacter par mail à <maxime@quandalle.com>.

### Générer un pdf

J'utilise [pandoc](http://johnmacfarlane.net/pandoc/) pour convertir les
fichiers markdown source en PDF :

```bash
$ sudo apt-get install pandoc texlive-lang-french
$ pandoc -S --chapters --table-of-contents --latex-engine=xelatex -V lang=french -o meteor.pdf 0*.md
```

### Générer un livre web

J'utilise [gitbook](https://www.gitbook.io/) pour générer un livre web :

```bash
$ npm install -g gitbook
$ gitbook serve
```
