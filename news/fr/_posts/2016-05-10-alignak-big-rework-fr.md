---
layout: post
title: Gros travail dans alignak
lang: en
ref: alignak
modified:
description: "Gros travail dans alignak"
tags: [alignak]
image:
  feature:
  credit:
  creditlink:
comments: true
share:
date: 2016-05-10T08:21:50+01:00
---

Sebastien Coavoux et David Durieux ont travaillés sans relâche ces dernières semaines dans Alignak et ont fait ces modifications :

* Utilisation de uuid dans tous les objets à la place des id
* Dissociation de presque tous les items : les attributs des objets ne sont plus des instances d'autres objets excepté pour quelques cas (commandcall / check_period et acknowledgemnt)
* Remplacement de pickle par une sérialisation manuellen : renvoi une représentation au format JSON des objets.
* Suppression de base64 / zlib de la sérialisation dans HTTP et utilisation des facilités de CherryPy pour avoir un json
* Modification des héritages de classe: l'instanciation des objets est plus générique
* Suppression des fonctions et classes inutilisés
* Augmentation des la queue de CherryPy pour les gros environnements


Vous pouvez vous demander, est-ce mieux?

OUI bien sûr!

Nous avons un code plus lisible, de meilleurs performances pour sérialiser et envoyer aux autres daemons (on gagne entre 10 et 50% de temps, CPU et mémoire).

Vous pourrez voir la différence dans la prochaine release d'Alignak ;)
