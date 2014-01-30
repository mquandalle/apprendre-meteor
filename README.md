# Apprendre Meteor

> **Attention :** La rédaction de ce cours n'est pas terminée.

Ce cours en français va vous permettre de découvrir le framework
[Meteor](https://www.meteor.com/). Vous pouvez accéder à la version PDF
[ici](https://github.com/mquandalle/apprendre-meteor/releases).

Ce cours est proposé sous la license
[Creative Commons BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.fr).

## Contribuer

Les contributions sont les bienvenues via les *Pull Request* sur Github.
Vous pouvez également me contacter par mail à <maxime@quandalle.com>.

### Générer un pdf

Nous utilisons [pandoc](http://johnmacfarlane.net/pandoc/) pour convertir les
fichiers markdown source en PDF.

```bash
pandoc -S --chapters --table-of-contents -V lang=french -o meteor.pdf 0*.md
```

[TODO] Générer également les formats "livre électronique" (EPUB...)
