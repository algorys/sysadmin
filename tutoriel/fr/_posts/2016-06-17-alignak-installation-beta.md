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
author: algorys mohierf
comments: true
share:
date: 2016-06-17T13:20:05+01:00
---

# WIP : Tutoriel en cours de correction

# Introduction

Alignak est une solution de monitoring. Celle-ci est utilisé pour superviser votre infrastructure et pour être informé lorsqu'il y a des problèmes et des dégradations de vos services (par mail, SMS, XMPP...). Alignak peut aussi bien être utilisé pour des petits structures que pour des plus importantes (Datacenter, fail-over, load-balancing).

**Attention :** Alignak est encore en développement et peut être instable ! Ce tutoriel est seulement pour ceux qui veulent tester ces applications.

Durant l'installation, vous allez installer les applicatifs suivants :

* Alignak : les démons d'Alignak.
* Alignak-Backend : le backend (Base de Données) d'Alignak.
* Alignak-Webui : l'interface Web d'Alignak.

Ces serveurs demandent un minimum de connaissance en Linux et d'être débrouillard.

Les installations qui suivent sont faites à partir des dépôts **git** (pour avoir les dernières corrections) mais, pour chacun d'entre eux, vous pouvez les installer à l'aide de `pip` ! Pour chaque étape, au lieu de `git` faites :

* `pip install alignak`
* `pip install alignak-backend`
* `pip install alignak-webui`

Les étapes d'installations ne sont donc pas nécessaires dans ce cas là (comme `sudo python setup.py install`).

**Soyez prudent, ne mélangez pas les types d'installation !**

Maintenant que vous êtes prévenus, vous pouvez commencez à jouer ;) !

# Prérequis

Pour tous ces applicatifs, vous aurez besoin des dépendances suivantes :

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

Vous devez ensuite créer un utilisateur _alignak_ et vous connecter avec :

```bash
sudo adduser alignak
# Ici on donne les droits sudo à alignak pour faciliter l'installation.
# Vous pouvez "jongler" avec un utilisateur root à la place.
sudo adduser alignak sudo
sudo su - alignak
```

# Alignak

# Installation et lancement d'Alignak

En premier, vous devez récupérer les sources d'Alignak. On vas essayer d'organiser notre installation, donc créez un dossier _repos_ avant :

```bash
cd ~; mkdir repos; cd repos
git clone https://github.com/Alignak-monitoring/alignak.git
cd alignak
```

Installez les dépendences avec _pip_ et lancer l'installation :

```bash
# Ne lancez pas "pip" avec sudo. Si vous rencontrez des problèmes avec l'installation, réessayez avec l'option '--user'.
pip install -r requirements.txt
sudo python setup.py install
```

Normalement vous devriez avoir la sortie suivante à la fin :

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
sudo chown -R alignak:alignak /usr/local/etc/alignak
```

Maintenant assurez-vous d'être connecté avec _alignak_ et lancez ses démons :

```bash
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

C'est bon, les démons d'Alignak sont démarrés !

## Configuration d'Alignak

Les fichiers de configuration d'Alignak sont situés dans le dossier */usr/local/etc/alignak*. Voici les dossiers principaux :

    - certs
    - daemons: daemons base configuration (communication, directories, ...)
    - arbiter_cfg:
      + daemons_cfg: configuration (modules, ...)
      + modules: modules configuration files
      + resource.d: resources configuration
      + objects: Nagios-like flat files configuration for monitored objects

# Plugins Nagios

Pour lancer des "checks" sur vos hôtes / services, Alignak a besoin de plugins. Le plus souvent, ceux de Nagios sont utilisés. Pour les installer, c'est très simple, téléchargez simplement l'archive du site officiel : [Nagios-plugins](https://nagios-plugins.org/).

```bash
cd ~; mkdir tools; cd tools
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
tar -xzvf nagios-plugins-2.1.1.tar.gz
# Maintenant installez-le
cd nagios-plugins-2.1.1/
./configure --with-nagios-user=alignak --with-nagios-group=alignak
make
sudo make install
```

Vos plugins sont installés. Ils ne seront peut-être pas isntallés dans le répertoire `/usr/lib/nagios/plugins`. Dans ce cas, cherchez où ils sont (`whereis nagios` par exemple) et éditez le fichier suivant `/usr/local/etc/alignak/arbiter_cfg/resource.d/paths.cfg` en conséquence :

```bash
vi /usr/local/etc/alignak/arbiter_cfg/resource.d/paths.cfg

    $NAGIOSPLUGINSDIR$=/usr/local/nagios/libexec
```

# Le Backend d'Alignak

## Prérequis du Backend

Vous devez installer `MongoDB` et `uwsgi` pour le faire tourner, donc on y va :

```bash
sudo apt-get install mongodb uwsgi uwsgi-plugin-python
```

## Récupérer les sources

Maintenant que nos démons tournent, on doit leur fournir le backend. Restez connecté en tant qu'utilisateur _alignak_ et tapez :

```bash
cd ~/repos
# Clonez dans le dossier backend
git clone https://github.com/Alignak-monitoring-contrib/alignak-backend.git backend
cd backend
```

Ensuite comme précédemment, on installe les prérequis et on lance l'installeur :

```bash
# Ne lancez pas "pip" avec sudo. Si vous rencontrez des problèmes avec l'installation, réessayez avec l'option '--user'.
pip install -r requirements.txt
sudo python setup.py install
```

## Lancement d'alignak-backend

Maintenant que le backend est installé, il faut le lancer.

Alignal-Backend est une application compatible avec **Python WSGI**. Comme cela, il est très pratique de'utiliser uWSGI comme lanceur.

Il existe plusieurs solutions :

### Créer votre "launcher"

Créez un dossier dédié :

```bash
cd ~; mkdir -p app/alignak-backend; cd app/alignak-backend
# Créez un lien symbolique, comme ça il sera mis à jour lors de la maj du dépôt
ln -s ~/repos/alignak-backend/alignak_backend.py ~/app/alignak-backend/alignakbackend.py
```

Ensuite lancez l'application comme suit :

```bash
# Remplacez xxx.xxx.xxx.xxx par l'IP de votre serveur
uwsgi --plugin python --wsgi-file alignakbackend.py --callable app --socket xxx.xxx.xxx.xxx:5000 --protocol=http --enable-threads
```

> **Soluces et Aides :** Si vous rencontrez des difficultés lors du lancement (comme l'import en Python), vérifiez que vous avez _MongoDB_ et le plugin _uwsgi-plugin-python_ d'installés. Verifiez les erreurs durant la mise en route. Supprimez les fichiers _*.pyc_ dans le répertoire courant. Essayez de vous déconnecter / reconnecter pour voir si il n'y a pas un problème de mise à jour des _paths_.

> **Note :** Il existe beaucoup de parametres pour configurer et optimiser uWSGI; consultez : [le projet uWSGI](http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html).

### Utilisez les scripts déjà prêts

Le dépôt du projet inclus un script d'example par défaut pour lancer l'application : `bin/run.sh`. Ce script inclus une ligne de commande par défaut qui démarre le backend et le met à l'écoute de toutes les interfaces, sur le port 5000.

Depuis le dossier racine du projet :

```bash
./bin/run.sh
```

# Modules du Backend d'Alignak

A présent, vous avez _alignak_ et son _backend_ qui tournent, mais les deux ne communiquent pas ensemble. Vous devez isntaller et configurer les modules pour les démons d'Alignak.

## Récupérer les sources

Restez connecter en tant qu'utilisateur _alignak_ et tapez les commandes suivantes :

```bash
cd ~/repos
# keep repos in backend
git clone https://github.com/Alignak-monitoring-contrib/alignak-module-backend.git backend-modules
cd backend-modules
```

Ensuite, vous connaissez la marche à suivre :

```bash
# Ne lancez pas "pip" avec sudo. Si vous rencontrez des problèmes avec l'installation, réessayez avec l'option '--user'.
pip install -r requirements.txt
sudo python setup.py install
```

## Configuration des modules

Le script d'installtion a créé des fichier de configuration dans votre dossier */usr/local/etc/alignak/arbiter_cfg/modules*.

Vous devez configurer l'URL de votre abckend et les informations de connexion pour les 3 fichiers suivants :

```bash
cd /usr/local/etc/alignak/arbiter_cfg
sudo vi modules/mod-alignakbackendarbit.cfg
sudo vi modules/mod-alignakbackendbrok.cfg
sudo vi modules/mod-alignakbackendsched.cfg
```

> **Conseil :** Actuellement, vous avez juste à décommentez les variables `username` et `password` pour permettre la conexion au backend. Vérifiez l'**api_url** dans ce fichier également !

Ensuite, vous devez configurer les démons d'Alignak poiur l'informer des modules existants :

> **Note :** Pour l'instant, le module du scheduler n'est pas fonctionnel, vous ne devez pas le configurer ! Des corrections arriveront bintôt sur la rétention de données...

```bash
cd /usr/local/etc/alignak/arbiter_cfg
sudo vi daemons_cfg/arbiter-master.cfg

    # Backend module
    modules      alignakbackendarbit


sudo vi daemons_cfg/broker-master.cfg

    # Backend module
    modules      alignakbackendbrok


sudo vi daemons_cfg/scheduler-master.cfg

    # Backend module
    modules      alignakbackendsched
```

Une fois cela fait, redémarrez les démons d'Alignak :

```bash
/usr/local/etc/init.d/alignak restart
```

> **Astuces :** Si des erreurs apparaissent, vous aurez des informations dans les fichiers de log situés dans */usr/local/var/log/alignak*.

# Outil d'import pour Alignak-Backend

Maintenant que tout tourne bien, vous devez remplir la base de donnée de votre backend avec les données des hôtes et des services que vous souhaitez monitorer. Le backend d'Alignak fournit une API REST que vous pouvez utilisez avec cUrl ou un autre outil (tel que Postman), mais qu'en est-il de vos fichiers de configuration de Nagios et comment les importer auto-magiquement ?

## Récupérer les sources

Restez toujours connecté en tant qu'_alignak_ et tapez :

```bash
cd ~/repos
# Clonez dans le dossier backend-import
git clone https://github.com/Alignak-monitoring-contrib/alignak-backend-import.git backend-import
cd backend-import
```

Ensuite, installez tout ce qui est nécessaire :

```bash
# Ne lancez pas "pip" avec sudo. Si vous rencontrez des problèmes avec l'installation, réessayez avec l'option '--user'.
pip install -r requirements.txt
sudo python setup.py install
```

## Lancer l'import

Alignak-backend-import a créé un script appelé `alignak_backend_import` dans votre dossier */usr/local/bin*.

Vous pouvez lancer cette commande avec plusieurs options. Tapez la commande suivante pour en savoir plus :

```bash
alignak_backend_import -h
```

Un exemple de configuration est disponible dans le répertoire du projet : *test/shinken_cfg_files/default*. Vous pouvez importer cette configuration avec la commande suivante :

```bash
alignak_backend_import -d -b http://127.0.0.1:5000 test/shinken_cfg_files/default/_main.cfg
```

Quelques explications :

- `-d` supprime les anciens éléments existants dans le backend
- `-b http://127.0.0.1:5000` specifie l'utilisation du backend local sur le port 5000
- `test ... _main.cfg` est le fichier plat de configuration à importer

> **En savoir plus :** Une documentation détaillée est disponible ici : [Documentation alignak-backend-import](http://alignak-backend-import.readthedocs.io/en/latest/index.html).


N'oubliez pas de redémarrer Alignak pour qu'il mette à jour votre nouvelle configuration :

```bash
/usr/local/etc/init.d/alignak restart
```

# Interface Web d'Alignak

A présent, vous avez _alignak_ et son _backend_ qui tournent. Vous pouvez installer l'interface Web ("Webui"). Allons-y :

```bash
cd ~/repos
git clone https://github.com/Alignak-monitoring-contrib/alignak-webui.git
cd alignak-webui
```

On repart pour une petite installation :

```bash
# N'oubliez pas l'option '--user' en cas de problèmes.
pip install -r requirements.txt
sudo python setup.py install
```

Ensuite, retournez dans votre dossier _app_ et copiez le fichier de configuration :

```bash
cd ~/app; mkdir webui; cd webui
cp ~/repos/webui/etc/settings.cfg settings.cfg
```

Le fichier principal de configuration se trouve dans */etc/alignak_webui* (ou */usr/local/etc/alignak-webui*). L'application lira ce fichier et remplacera ses varaibles par celle trouvées dans *./settings.cfg* et *./etc/settings.cfg* si ils existent !

Vous pouvez définir différentes configuration dans _settings.cfg_ mais vous devez définir l'URL de votre backend :

```conf
# settings.cfg
alignak_backend = http://127.0.01:5000
```

# Lancer alignak-webui

Maintenant que tout est prêt, vous devez démarrer l'application Web.

Comme le backend d'Alignak, la WebUI utilise Python WSGI et il existe différentes solutions pour le lancer...

### Créer votre propre "launcher"

Retournez dans votre dossier _~/app/webui_ :

```bash
# create symlink, like this update when updating repos
ln -s ~/repos/webui/bin/run.sh ~/app/webui/run.sh
cp ~/repos/webui/bin/alignak_webui.py alignakwebui.py
```

Ensuite lancez votre application comme suit :

```bash
# Remplacez xxx.xxx.xxx.xxx par l'IP de votre serveur
uwsgi --plugin python --wsgi-file alignakwebui.py --callable app --socket xxx.xxx.xxx.xxx:5001 --protocol=http --enable-threads
```

> **Soluces et astuces :** Si vous rencontre les problèmes, verifiez que vous avez bien installé _MongoDB_ et _uwsgi-plugin-python_. Regarder bien vos logs ou les erreurs durant le lancement. Essayez d'effacer les fichiers _*.pyc_ dans le dossiers courant.

> **Note :** Vous pouvez en savoir plus sur uWSGI et sa configuration sur : [Projet uWSGI](http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html).

### Utilisez les scripts fournis

Le dépôt du projet inclus un fichier par défaut pur démarrer votre WebUI : `bin/run.sh`. Ce script inclus une commande par défaut qui écoute sur toutes les interfaces, sur le port 5001.

Depuis la racine du dépôt :

```bash
./bin/run.sh
```

## Utilisez la WebUI

Vous pouvez atteindre votre interface Web sur : http://xxx.xxx.xxx.xxx:5001 où _xxx.xxx.xxx.xxx_ est l'IP que vous avez renseigné pour votre WebUI.

Vous pouvez vous connecter avec les informations suivantes :

* username: admin
* password: admin

L'application crée un fichier de log. La configuration de ce log est défini dans votre fichier _settings.cfg_, dans la section `logs`. Si l'utilisateur le permet, il est stocké dans le dossier _/var/log/alignak-webui_ (ou _/usr/local/var/log/alignak-webui_); sinon, le fichier peut-être parcouru dans le dossier courant :

```bash
tail -f ~/app/webui/alignak-webui.log
```

## VM-Test

Si votre installation est sur une machine virtuelle (avec VMWare par exemple) et que vous avez utilisez la boucle locale (`127.0.0.1:5001`) comme adresse par défaut, votre WebUI ne sera pas accessible par votre navigateur. En effet, votre boucle locale est différente de celle de votre serveur de test.

Mais vous pouvez tricher en faisant croire que la boucle locale du serveur est la votre (vive Linux !!). Ouvrez un terminal et tapez la commande suivante :

```bash
ssh -L 5001:127.0.0.1:5001 login@ip_vm_test
```

Où *ip_vm_test* est l'adresse IP de votre serveur de test (et  non pas sa boucle locale !). Ensuite réessayez d'atteindre [http://127.0.0.1:5001](http://127.0.0.1:5001) dans votre navigateur.

# Outils complémentaires

Il existe d'autres outils pour compléter la Suite d'Alignak. Voici la liste :

* [Alignak-App](http://alignak-app.readthedocs.io/en/develop/index.html) : App Indicator pour Alignak qui affiche des notifications si quelque chose ne va pas.

# WIP


