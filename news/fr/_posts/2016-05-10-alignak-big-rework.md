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

Sebastien Coavoux et David Durieux ont travaill�s sans rel�che ces derni�res semaines dans Alignak et ont fait ces modifications :

* Utilisation de uuid dans tous les objets � la place des id
* Dissociation de presque tous les items : les attributs des objets ne sont plus des instances d'autres objets except� pour quelques cas (commandcall / check_period et acknowledgemnt)
* Remplacement de pickle par une s�rialisation manuellen : renvoi une repr�sentation au format JSON des objets.
* Suppression de base64 / zlib de la s�rialisation dans HTTP et utilisation des facilit�s de CherryPy pour avoir un json
* Modification des h�ritages de classe: l'instanciation des objets est plus g�n�rique
* Suppression des fonctions et classes inutilis�s
* Augmentation des la queue de CherryPy pour les gros environnements


Vous pouvez vous demander, est-ce mieux?

OUI bien s�r!

Nous avons un code plus lisible, de meilleurs performances pour s�rialiser et envoyer aux autres daemons (on gagne entre 10 et 50% de temps, CPU et m�moire).

Vous pourrez voir la diff�rence dans la prochaine release d'Alignak ;)
