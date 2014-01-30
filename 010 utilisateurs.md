# Accueillons nos utilisateurs

## Mise en place d'un système de gestion des utilisateurs

Le package de base pour la gestion des comptes utilisateurs s'appele `accounts-base`. En général il est implicitement ajouté via l'utilisation d'un des packages plus avancé comme `accounts-password`, `accounts-facebook`, `accounts-twitter`, etc.

`Meteor.users` est une simplement une collection particulière dédiée par convention aux données des utilisateurs. Cette collection contient un document par utilisateur dans lequel vous êtes libre de définir tous les champs désirés. Certains champs ont cependant un comportement par défaut particulier :

[TODO] Liste de la doc

## Connexion via Facebook, Twitter, LinkedIn et quelques autres
