---
layout: post
title: Installer Shinken
lang: fr
ref: shinken
modified:
description: Comment installer Shinken
tags: [tutoriel, shinken, monitoring]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-03-08T10:00:05+01:00
---

# Introduction

Dans ce tutoriel, nous allons voir comment installer [Shinken](http://www.shinken-monitoring.org/index.php) sur un serveur Linux. En soit l'installation de Shinken n'est vraiment pas difficile et peut être faites de plusieurs manières. La [Documentation](http://shinken.readthedocs.org/en/latest/index.html) est vraiment très bien faites et agréable à lire. Toutefois, ce tutoriel va montrer l'installation via les sources et notamment la façon dont on peut configurer Shinken et son interface Web.

# Les différentes installations de Shinken

Si vous êtes allé faire un tour sur la documentation, vous avez pu voir qu'il existe 3 manières différentes d'installer Shinken : 

* via [Pip](https://pip.pypa.io/en/stable/) (l'installateur de modules de Python) qui est souvent à jour.
* via les paquets qui ne sont parfois pas à jour
* via les sources du [dépôt Shinken](https://github.com/naparuba/shinken) que nous allons utiliser ici.

Libre à vous d'installer Shinken comme bon vous semble. Mais le **plus important** est de n'utiliser qu'**une seule manière** pour l'installation et les mises à jour de Shinken ! En aucun cas, vous devez mélanger plusieurs type d'installations sous peine de voir votre serveur planter, avoir des bugs ou ne plus marcher du tout ! Je vous invite d'ailleurs de vous faire un petit fichier dans votre installation pour noter comment vous avez installé Shinken et ses différents modules / librairies.

# Prérequis

Shinken est écrit en Python, il aura donc besoin d'une version de Python pour fonctionner (version 2.6 ou supérieur et si possible 2.7 pour de meilleurs performances). Il aura aussi besoin de [python-pycurl](http://pycurl.io/) et de [setuptools](https://pypi.python.org/pypi/setuptools/).

Voici les différentes étapes pour ces dépendances, en sachant que ce tutoriel a été fait sur un Ubuntu Server 14.04.4 LTS.

## Installer Python

Pour installer Python, rien de plus simple :

```bash
sudo apt-get update
sudo apt-get install python
```

En général, sur Ubuntu Server, Python 2.7 est déjà installé par défaut. Il est possible que sur d'autres distributions il ne soit pas disponible via les dépôts. Renseignez-vous alors sur le site de votre distribution.

### Installer PyCurl

Pour installer PyCurl, il suffit de l'installer via les paquets :

```bash
sudo apt-get install python-pycurl
```

### Installer setuptools

Pour installer SetupTools, la démarche est la même :

```bash
sudo apt-get install python-setuptools
```

Voilà, vous avez normalement toutes les dépendances requises pour l'installation de Shinken.

# Récupération des sources de Shinken

Pour récupérer les sources de Shinken et surtout la dernière _release_, rendez-vous sur le [dépôt de Shinken](https://github.com/naparuba/shinken) et téléchargez-la :


```bash
sudo apt-get install -y git
cd ~
git clone https://github.com/naparuba/shinken.git 
cd shinken
# Vous pouvez voir les différentes release en tapant "git tag -l"
git checkout 2.4.2
```

Voilà, votre dossier Shinken est prêt, vous allez pouvoir lancer l'installation.

> Vous pouvez bien évidemment la télécharger avec `wget` ou via une interface graphique, mais je préfère cette méthode, car elle permettra de mettre à jour plus facilement votre version de Shinken. Avec Git vous n'aurez qu'à "tirer" les nouvelles versions et vous remettre sur un `tag` plus récent.
 
# Installer Shinken

Vous allez maintenant pouvoir lancer l'installation de Shinken avec Python. Mais pour cela, il aura besoin d'un utilisateur `shinken`. Crééz-le avant de lancer le setup :

```bash
adduser shinken
```

Une fois cela fait, vous pouvez lancer la commande d'installation :

```bash
cd ~/shinken
sudo python setup.py install
```

Normalement l'installation devrait se dérouler sans problème. Vous aurez quelques `warning` au début pour signaler qu'il n'est pas sous Windows, etc... mais surtout à la fin de l'installation vous aurez les lignes suivantes :

```bash
Finished processing dependencies for Shinken==2.4.2
Changing owner of /etc/shinken to shinken:shinken
Changing owner of /var/run/shinken to shinken:shinken
Changing owner of /var/log/shinken to shinken:shinken
Changing owner of /var/lib/shinken/ to shinken:shinken
Changing owner of /var/lib/shinken/libexec to shinken:shinken
Changing owner of /usr/bin/shinken-receiver to shinken:shinken
Changing owner of /usr/bin/shinken-discovery to shinken:shinken
Changing owner of /usr/bin/shinken-arbiter to shinken:shinken
Changing owner of /usr/bin/shinken-poller to shinken:shinken
Changing owner of /usr/bin/shinken-broker to shinken:shinken
Changing owner of /usr/bin/shinken-scheduler to shinken:shinken
Changing owner of /usr/bin/shinken to shinken:shinken
Changing owner of /usr/bin/shinken-reactionner to shinken:shinken
Notice: for better performances for the daemons communication, you should install the python-cherrypy3 lib
Shinken setup done
```

Comme vous pouvez le voir, Shinken a donné les droits à l'utilisateur `shinken` que vous avez créé au préalable, sur les dossiers dont il a besoin. Vous n'avez donc rien à faire de ce côté-ci.

Il vous signale aussi que les `démons` (`daemons` en anglais) de Shinken seront plus performants si vous installer la librairie `python-cherrypy3`. Cette dépendance est optionnelle donc libre à vous de l'installer ou pas :

```bash
sudo apt-get install -y python-cherrypy3
```

Voilà, Shinken est prêt à fonctionner. Vous pouvez l'activer et le démarrer :

```bash
sudo update-rc.d shinken defaults
sudo service shinken start
```

Vous devriez avoir la sortie suivante :

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

Félicitation, votre Shinken est maintenant opérationnel et tous ses démons sont lancés. Vous allez pouvoir monitorer les différents serveurs de votre parc. Mais pour rendre cela plus pratique, il serait bien d'installer une interface Web.

# Installer l'interface Web

Pour que vous puissiez voir d'une manière plus agréable si vos hôtes sont bien monitorés, une interface Web est disponible pour Shinken. Il existe 2 versions de cette interface :

* webui : plus trop maintenue par les développeurs, cette interface est amenée à disparaître. Mais elle est toujours fonctionnelle.
* webui2 : plus à jour, ergonomique et en plein développement. Elle n'est pas encore complète mais possède déjà tout ce qu'il faut pour être fonctionnelle.

Connectez vous en tant qu'utilisateur **shinken** :

```bash
sudo su - shinken
```

Et initialisez la CLI de Shinken pour générer le fichier `.ini` contenant les chemins vers les différents répertoires de configuration :

```bash
shinken --init
```
> **Important :** toutes les commandes `shinken` doivent être lancées en tant qu'utilisateur `shinken` !

Vous devriez avoir la sortie suivante :

```bash
Creating ini section paths
Creating ini section shinken.io
Saving the new configuration file /home/shinken/.shinken.ini
```

Maintenant vous pouvez installer l'interface que vous souhaitez avoir. 

> Attention, même si les deux `webui` peuvent cohabiter, je vous conseille de n'en choisir qu'une, afin de ne pas vous mélanger les pinceaux. Si vous installez les deux interfaces, mettez un port différent pour l'une d'entre elle (autre que le `7767` !).

## Cas 1 : Interface Webui

Pour l'interface Webui, il va falloir installer d'autres modules. Tapez les commandes suivantes pour les installer :

* `shinken install webui` -> l'interface webui.
* `shinken install auth-cfg-password` : le module d'authentification de base. (D'autres modules d'authentification existent)
* `shinken install sqlitedb` : le module pour stocker les données des utilisateurs.

En tapant chaque commande vous devriez avoir la sortie :

```bash
Grabbing : module_name
OK module_name
```

Une fois les modules installés, vous allez devoir indiquer à Shinken que vous voulez les utiliser.

Ouvrez le fichier `/etc/shinken/brokers/broker-master.cfg` et trouvez la ligne non commentée où il y a `modules` pour y ajouter **webui** :

```conf
[...]
modules    webui
[...]
```

Éditez ensuite le fichier `/etc/shinken/modules/webui.cfg` de la même manière pour y ajouter **auth-cfg-password** et **SQLitedb** :

```conf
[...]
modules     auth-cfg-pasword,SQLitedb
[...]
```

Félicitations ! Votre interface **Webui** est configurée. Si tous les démons de Shinken ont bien été redémarrés, vous devriez voir la page suivante en vous rendant sur l'adresse `http://ip_serveur:7767` :

<figure>
    <img src="{{ site.url }}/images/shinken/shinken_webui.png" alt="">
    <figcaption>Shinken - Écran de connexion Webui</figcaption>
</figure>

## Cas 2 : Interface Webui2

Pour l'interface Webui2 l'installation est différente car elle gère elle-même pas mal de choses par défaut. Vous allez devoir installer quelques librairies de Python supplémentaires, pour qu'elle puisse fonctionner.

En tant qu'utilisateur Shinken, installez `Webui2` :

```bash
shinken install webui2
```

Ensuite , en tant qu'utilisateur `root`, installez le programme `pip` et le système de gestion de base de donnée `mongodb` :

```
sudo apt-get install python-pip mongodb
```

Vous pouvez installer les librairies suivantes via les paquets de votre distribution, mais vous n'aurez certainement pas les versions requises pour `webui2`. Vous devez donc les installer avec `pip` :

```
sudo pip install pymongo>=3.0.3 requests arrow bottle==0.12.8
```

Une fois toutes ces dépendances installées, vous devez indiquer à Shinken le module `webui2` dans le _broker_ (`vi /etc/shinken/brokers/broker-master.cfg`) :

```bash
[...]
modules     webui2
[...]
```

Redémarrez maintenant Shinken :

```bash
sudo service shinken restart
```

Félicitations ! Votre interface **Webui2** est configurée. Si vous vous rendez sur `http://ip_serveur:7767` avec votre navigateur, vous devriez voir la page suivante :

<figure>
    <img src="{{ site.url }}/images/shinken/shinken_webui2.png" alt="">
    <figcaption>Shinken - Écran de connexion Webui2</figcaption>
</figure>

# Configurer l'utilisateur Admin

Une fois toutes ces manipulations effectuées, vous pouvez aller changer le mot de passe par défaut du compte `admin` dans le fichier `/etc/shinken/contacts/admin.cfg` :

```conf
define contact{
    use             generic-contact
    contact_name    admin
    email           your@email.com
    pager           0600000000   ; contact phone number
    password        your_password
    is_admin        1
    expert          1
}
```

Enregistrez et quittez.

Vous pouvez vérifier vos configurations en tapant la commande suivante (en étant utilisateur `sudo` ou `root`) :

```bash
sudo service shinken check
```

Redémarrez les services Shinken pour prendre en compte la nouvelle configuration :

```bash
sudo service shinken restart
```

Et connectez vous sur l'interface Web avec votre identifiant et le nouveau mot de passe.

# Commandes et Nagios Plugins

Comme vous pouvez le voir sur l'interface web (ou dans les logs du `scheduler` de Shinken), le seul _hôte_ présent s'appelle **localhost**. Vous allez peut-être même voir l'erreur suivante :

```bash
[Errno 2] No such file or directory
```

Et l'hôte sera signalé `DOWN`, c'est à dire que Shinken vous indique que l'hôte est "tombé" ! Pourtant votre serveur fonctionne parfaitement et fait tourner Shinken !

C'est en fait normal car il vous manque quelque chose de très important : des commandes de `check` ! Et si vous n'avez pas installer ce qu'il faut, Shinken ne trouvera pas la commande à exécuter, et marquera donc l'hôte `DOWN`.

## Les commandes et les "paths"

En effet Shinken va avoir besoin de commandes, fonctionnant sur le protocole [snmp](https://fr.wikipedia.org/wiki/Simple_Network_Management_Protocol) pour la plupart, qui vont lui permettre de monitorer vos serveurs.

Pour déjà comprendre le fonctionnement de Shinken, ouvrez le fichier suivant (en tant que l'utilisateur `shinken`) :

```bash
vi /etc/shinken/commands/check_host_alive.cfg
```

Vous devriez avoir quelque chose de similaire à :

```conf
define command {
    command_name    check_host_alive
    command_line    $NAGIOSPLUGINSDIR$/check_ping -H $HOSTADDRESS$ -w 1000,100% -c 3000,100% -p 1
}
```

Cela indique que la commande `check_host_alive` est définie comme suit :

* Son Nom : `check_host_alive` (c'est ce qui sera affiché dans les logs ou sur l'interface web)
* La commande : définie ici par une variable `$NAGIOSPLUGINSDIR$` et une commande `check_ping` avec des paramètres...

Vous avez donc toutes les informations pour savoir ce que fait la commande... mais par contre il vous manque la définition de la variable `$NAGIOSPLUGINSDIR$` ! 

En fait, elle est définie dans le fichier `/etc/shinken/resource.d/paths.cfg` (ouvrez-le) :

```config
# Nagios legacy macros
$USER1$=$NAGIOSPLUGINSDIR$
$NAGIOSPLUGINSDIR$=/usr/lib/nagios/plugins

#-- Location of the plugins for Shinken
$PLUGINSDIR$=/var/lib/shinken/libexec
```

Comme vous pouvez le voir, par défaut, Shinken s'attend à avoir des commandes dans le dossier `/usr/lib/nagios/plugins` ! Mais vu que pour le moment ce dossier n'existe pas et qu'il n'y a pas de commande / fichier `check_ping`, il vous le signale en mettant une erreur (`[Errno 2] No such file or directory`).

Vous allez donc devoir installer les plugins Nagios.

## Installer les plugins Nagios

Pour installer les plugins Nagios, c'est très simple il suffit d'aller les télécharger sur le site officiel : [Nagios-plugins](https://nagios-plugins.org/).

```bash
cd ~
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
tar -xzvf nagios-plugins-2.1.1.tar.gz
```

Maitenant vous allez devoir les installer :

```bash
cd nagios-plugins-2.1.1/
./configure --with-nagios-user=shinken --with-nagios-group=shinken
make
sudo make install
```

Et voilà, normalement vos plugins sont installés. Il est possible qu'ils ne soient pas installés dans le répertoire `/usr/lib/nagios/plugins`. Dans ce cas là, faites une recherche pour savoir où ils se trouvent (`whereis nagios` par exemple) et modifiez le fichier `/etc/shinken/resource.d/paths.cfg` en conséquence.

Par exemple :

```conf
# Nagios legacy macros
$USER1$=$NAGIOSPLUGINSDIR$
#$NAGIOSPLUGINSDIR$=/usr/lib/nagios/plugins
$NAGIOSPLUGINSDIR$=/usr/local/nagios/libexec

#-- Location of the plugins for Shinken
$PLUGINSDIR$=/var/lib/shinken/libexec
```

Vous n'avez ensuite plus qu'à redémarrer Shinken pour appliquer ces changements :

```bash
sudo service shinken restart
```

Allez voir sur l'interface Web une fois tous les démons redémarrés. Et là normalement votre hôte `localhost` est bien `UP` (il y aura peut-être un petit délai le temps que Shinken relance bien tous ses démons) !

Félicitations ! Vous avez enfin réussi à monitorer un serveur avec Shinken !

# Conclusion

Shinken est assez facile à mettre en place et demande juste un peu d'organisation lors de l'installation de ses modules. Vous pouvez d'ailleurs retrouver une liste de ces modules sur le [site officiel](http://shinken.io/browse/modules/updated) avec leurs différentes configurations. Shinken est aussi capable d'afficher des graphiques sur son interface Web ou bien sur [GLPI](http://glpi-project.org/) grâce au plugin [glpi_monitoring](https://github.com/ddurieux/glpi_monitoring). La communauté est très active et dispose de nombreux développeurs et forums pour vous aider.
