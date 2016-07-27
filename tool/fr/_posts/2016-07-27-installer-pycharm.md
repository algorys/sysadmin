---
layout: post
title: Installer PyCharm sur Linux
lang: fr
ref: pycharm
modified:
description: Comment installer Pycharm sur Linux
tags: [tutoriel, python]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-07-27T09:00:05+01:00
---

# Installation

Pour ceux qui souhaitent contribuer à un projet ou développer en Python, PyCharm est un très bon IDE. Il possèdent beaucoup de fonctionnalités, comme le support de plusieurs langages (Python, Markdonw, Yaml, etc.) et de nombreux outils pour vous aider à développer votre librairie ou votre programme en python.

# Prérequis

Vous avez juste besoin d'une version de python de 2.4 à 3.5. Normalement, vos dépôt devraient la fournir:

```bash
# Example sur Debian pour python 2.7
sudo apt-get install python2.7
```

Vous devez avoir aussi un accès **root**.

# Télécharger l'archive

En premier, vous devez télécharger l'archive [ici](https://www.jetbrains.com/pycharm/download/#section=linux). Si vous n'avez pas de licence, téléchargez la **Community Edition**.

Vous récupererez une archive `tar.gz`. Une fois terminé, décompressez-la :

```bash
tar xzvf pycharm-community-2016.2.tar.gz -C /tmp/
```

# Supprimer l'ancienne installation

Si vous avez déjà PyCharm d'installé sur votre PC, supprimer l'ancien dossier et les liens symboliques :

```bash
sudo rm -R  /opt/pycharm-community
sudo rm /usr/local/bin/pycharm
sudo rm /usr/local/bin/inspect
```

# Installer PyCharm

Vous avez maintenant un dossier nommé **pycharm-community-2016.2** dans `/tmp`. Déplacez-le dans `/opt` :

```bash
sudo mv /tmp/pycharm-community* /opt/pycharm-community
```

# Création des Liens Symboliques

Maintenant, créez les liens nécessaire pour les utilisateurs:

```bash
sudo su -c "ln -s /opt/pycharm-community/bin/pycharm.sh /usr/local/bin/pycharm"
sudo su -c "ln -s /opt/pycharm-community/bin/inspect.sh /usr/local/bin/inspect"
```

# Lancer PyCharù

Voilà, c'est bon ! Vous avez juste à lancer PyCharm :

```bash
pycharm
```

Si vous aviez précédemment installer PyCharm, il essaiera d'importer vos anciennes configuration ("I want to import my settings from a previous version (~/.PyCharm2016.1/config)". Cliquez juste sur **OK**.

PyCharm est lancé, il mettra ensuite à jour vos plugins. Redémarrez-le après cette opération.

# Conclusion

Installer PyCharm est vraiment très facile et si vous développer en Python, c'est l'un des meilleurs IDE que vous trouverez sur Linux.
