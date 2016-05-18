---
layout: post
title: FusionInventory et les miroirs pour le d�ploiement d'applications
lang: fr
ref: fusioninventory
modified:
description: Comment utiliser les miroirs pour le d�ploiement d'application avec FusionInventory
tags: [tutorial, fusioninventory, glpi]
image:
  feature:
  credit:
  creditlink:
comments: true
share:
date: 2016-05-18T19:08:05+01:00
---

# D�ployer avec les miroirs

Vous pouvez d�ployer avec le p2p, mais si vous ne pouvez pas / voulez pas utiliser �a sur dans une architecture multi-sites, 
vous pouvez utiliser des mirois.


## Qu'est ce qu'un miroir?

Un miroir est un serveur HTTP utilis� par un agent qui va essayer de r�cup�rer le(s) fichier(s) requis pour le d�ploiement d'applications.


## Comment �a fonctionne ?


### Dans GLPI

Dans GLPI, ajouter un miroir dans le menu *Plugins* > *FusionInventory* > *D�ployer* > *Serveurs miroirs*

Dans le champ *Adresse du serveur miroir* l'url de votre serveur HTTP sur le site distant (Bien penser � cr�er le serveur miroir dans la bonne entit�).

Par exemple:

```
http://192.168.43.1/
```

### Les miroirs

Installer votre serveur HTTP favori (apache, nginx...) sur le site distant.

Copier / synchronizer le dossier (et les sous-dossiers) *glpi/files/_plugins/fusioninventory/files/repository* dans le serveur HTTP (� la racine du serveur dans notre exemple).


### L'agent

L'agent va recevoir une liste des serveurs miroir de GLPI.

Il va essayer chacun ce serveur (ces serveurs s'il y en a plusieurs de d�finis) avec le(s) fichier(s) requis pour le d�ploiement d'application. S'il ne le trouve pas, � la fin il va le r�cup�rer sur GLPI.


