---
layout: post
title: Gros travail dans alignak
lang: fr
ref: alignak-bigrework
modified:
description: "Gros travail dans alignak"
tags: [alignak]
image:
  feature:
  credit:
  creditlink:
author: ddurieux
comments: true
share:
date: 2016-05-10T08:21:50+01:00
---

Sebastien Coavoux et David Durieux ont travaillé sans relache ces dernières semaines dans Alignak et ont fait ces modifications :

* Utilisation de uuid dans tous les objets à la place des id
* Dissociation de presque tous les items : les attributs des objets ne sont plus des instances d'autres objets, exepté pour quelques cas (commandcall / chack_period et acknowledgemnt)
* Remplacement de pickle par une sérialisation manuelle : renvoi une représentation au format JSON des objets.
* Suppression de base64 / zlib de la sérialisation dans HTTP et utilisation des facilités de CherryPy pour avoir un json
* Modification des héritages de classe : l'instanciation des objets est plus générique
* Suppression des fonctions et classes inutilisées
* Augmentation de la queue de CherryPy pour les gros environnements

Vous pouvez vous demander, est-ce mieux ?

OUI bien sur !

Nous avons un code plus lisible, de meilleurs performances pour sérialiser et envoyer aux autres daemons (on gagne entre 10 et 50% de temps, CPU et mémoire).

Vous pourrez voir la différence dans la prochaine release d'Alignak ;)
