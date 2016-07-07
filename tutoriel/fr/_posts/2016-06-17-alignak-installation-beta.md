---
layout: post
title: Installer Alignak en Beta
lang: fr
ref: alignak-beta
modified:
description: Comment installer la Suite Alignak
tags: [tutoriel, alignak, monitoring]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-06-17T13:20:05+01:00
---

# WIP : Tutoriel en cours de correction

# Introduction

Alignak est une solution de monitoring. Celle-ci est utilisé pour superviser votre infrastructure et pour être informé lorsqu'il y a des problèmes et des dégradations de vos services (par mail, SMS, XMPP...). Alignak peut être aussi bien utilisé pour de petits environnement que pour de plus importants (Datacenter, fail-over, load-balancing).

**Attention :** Alignak est encore en développement et peut être instable ! Ce tutoriel est seulement pour ceux qui veulent tester ce serveur.

Durant l'installation, vous allez installer les applicatifs suivants :

* Alignak : les démons d'Alignak.
* Alignak-Backend : le backend d'Alignak.
* Alignak-Webui : l'interface Web d'Alignak.

Ces serveurs demandent un minimum de connaissance en Linux et d'être débrouillard.

Les installations qui suivent sont faites à partir des dépôts **git** (pour avoir les dernières corrections) mais pour chaque, vous pouvez les installer à l'aide de `pip` ! Pour chaque étape, au lieu de `git` faites :

* `pip install alignak`
* `pip install alignak-backend`
* `pip install alignak-webui`

Les étapes d'installations ne sont donc pas non plus nécessaires (comme `sudo python setup.py install`).

**Soyez prudent, ne mélangez pas les types d'installation !**

Maintenant que vous êtes prévenus, vous pouvez commencez à jouer ;) !

# Prérequis

Pour tous ces dépôts, vous aurez besoin des dépendances suivantes :

* Un server à jour : `sudo apt-get update && sudo apt-get upgrade`
* Python (**2.7** est recommandé) et des librairies : `sudo apt-get install python2.7 python2.7-dev python-pip`
* Git : `sudo apt-get install git`

Vérifiez votre version de Python :

```bash
$ python --version
# Output:
Python 2.7.6
```

**Astuce :** Vous pouvez mettre à jour `setuptools`, `pip` et `pbr` si jamais vous rencontrez des problèmes avec `pip install`.

Vous devez ensuite créer un utilisateur _alignak_ :

`sudo adduser alignak`

# Démons d'Alignak

En premier, vous devez récupérer les sources d'Alignak :

```bash
cd ~; mkdir repos; cd repos
git clone https://github.com/Alignak-monitoring/alignak.git
cd alignak
```

Installez les dépendences avec _pip_ et lancer l'installation :

```bash
sudo pip install -r requirements.txt
sudo python setup.py install
```

Normalement vous devriez avoir à la fin la sortie suivante :

```bash
=======================================================================================================
==                                                                                                   ==
==  Don't forget to create user and group 'alignak' or change daemons configuration                  ==
==                                                                                                   ==
=======================================================================================================

running install_egg_info
Copying alignak.egg-info to /usr/local/lib/python2.7/dist-packages/alignak-0.2.egg-info
running install_scripts
Installing alignak-broker script to /usr/local/bin
Installing alignak-scheduler script to /usr/local/bin
Installing alignak-receiver script to /usr/local/bin
Installing alignak-poller script to /usr/local/bin
Installing alignak-reactionner script to /usr/local/bin
Installing alignak-arbiter script to /usr/local/bin
```

Si vous n'avez pas créé l'utilisateur _alignak_ , **faites-le maintenant** !

Vous devez ensuite donner les droits sur certains dossiers :

```bash
sudo chown -R alignak:alignak /usr/local/var/run/alignak
sudo chown -R alignak:alignak /usr/local/var/log/alignak
```

Maintenant connectez-vous avec _alignak_ et lancez ses démons :

```bash
sudo su - alignak
/usr/local/etc/init.d/alignak start
```

Normalement, vous devriez avoir le texte suivant  dans votre terminal :

```bash
Starting scheduler:
   ...done.
Starting poller:
   ...done.
Starting reactionner:
   ...done.
Starting broker:
   ...done.
Starting receiver:
   ...done.
Starting arbiter:
   ...done.
```

C'est bon, les démons d'Alignak sont démarrés.

# Le Backend d'Alignak

## Prérequis du Backend

Vous devez installer `MongoDB` et `uwsgi` pour le faire tourner, donc on y va :

```bash
sudo apt-get install mongodb uwsgi uwsgi-plugin-python
```

## Récupérer les sources

Maintenant que nos démons tournent, on doit leur fournir le backend. Retournez dans votre répertoire et clonez :

```bash
cd ~/repos
git clone https://github.com/Alignak-monitoring-contrib/alignak-backend.git
cd alignak-backend
```

Ensuite comme précédemment, on installe :

```bash
sudo pip install -r requirements.txt
sudo python setup.py install
```

## Lancement d'alignak-backend

Pour lancer le _backend_, vous allez créer un répertoire dédié :

```bash
cd ~; mkdir -p app/alignak-backend; cd app/alignak-backend
# Créez un lien symbolique, comme ça il sera mis à jour lors de la maj du dépôt
ln -s ~/repos/alignak-backend/alignak_backend.py ~/app/alignak-backend/alignakbackend.py
```

Ensuite lancez l'application comme suit :

```bash
# Remplacez xxx.xxx.xxx.xxx par l'IP de votre serveur
sudo uwsgi --wsgi-file alignakbackend.py --callable app --socket xxx.xxx.xxx.xxx:5000 --protocol=http --enable-threads
```

> **Soluces et Aides :** Si vous rencontrez des difficultés lors du lancement (comme l'import en Python), vérifiez que vous avez _MongoDB_ et le plugin _uwsgi-plugin-python_ d'installés. Verifiez les erreurs durant la mise en route. Supprimez les fichiers _*.pyc_ dans le répertoire courant. Essayez de vous déconnecter / connceter pour voir si il n'y a pas un problème de mise à jour des _paths_.

# Interface Web d'Alignak

A présent, vous avez _alignak_ et son _backend_ qui tournent. Vous pouvez installer l'interface Web ("Webui"). Allons-y :

```bash
cd ~/repos
git clone https://github.com/Alignak-monitoring-contrib/alignak-webui.git
cd alignak-webui
```

On repart pour une petite installation :

```bash
sudo pip install -r requirements.txt
sudo python setup.py install
```

Ensuite, retournez dans votre dossier _app_ :

```bash
cd ~/app; mkdir alignak-webui; cd alignak-webui
ln -s ~/repos/alignak-webui/alignak_webui/app.py ~/app/alignak-webui/app.py
# TODO : A re-tester
# cp ~/repos/alignak-webui/etc/settings.cfg settings.cfg
```

Vous pouvez définir différentes configuration dans _settings.cfg_.

En lançant `sudo python app.py -h` vous pouvez avoir pas mal d'informations. Maitnenant lancez-le comme suit :

```bash
# Remplacez xxx.xxx.xxx.xxx par l'IP de votre serveur ou son FQDN.
sudo python app.py -n xxx.xxx.xxx.xxx -p 80 &
# Dans le cas où la Webui ne prend pas le bon serveur mais celui par défaut pour le backend
sudo python app.py -b http://xxx.xxx.xxx.xxx:5000 -n xxx.xxx.xxx.xxx -p 80 &
```

Maintenant votre webui est démarrée et vous pouvez allez sur : http://xxx.xxx.xxx.xxx/login dans votre navigateur. Les identifiants par défaut sont :

* username : admin
* password : admin

Vous pouvez également regarder les logs de celle-ci :

```bash
sudo tail -f ~/app/alignak-webui/alignak-webui.log
```

# WIP

