---
layout: post
title: Installer GLPI
modified:
description: Comment installer glpi
tags: [tutoriel, glpi]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2016-03-07T16:52:05+01:00
---

# Introduction

[GLPI](http://www.glpi-project.org/) est un gestionnaire de Parc Informatique qui permet de gérer les ressources de son parc, que ce soit au niveau matériel, logiciel ou bien réseau. Il permet, en outre, grâce à ses différents plugins, de s'adapter à quasiment tout type de parc informatique. De plus GLPI est très facile à installer, il peut discuter avec d'autres serveurs pour collecter automatiquement ses données ([OCS](http://www.ocsinventory-ng.org/fr/), [Fusion-Inventory](http://fusioninventory.org/)) et peut même faire du monitoring en se couplant avec un serveur [Shinken](http://www.shinken-monitoring.org/) (ou [Nagios](https://www.nagios.org/)).

# Prérequis

Pour installer GLPI, vous devez avoir certains prérequis :

* PHP 5.3 ou supérieur : `sudo apt-get install php5`
* Une base MySQL : `sudo apt-get install mysql-server`
* un serveur Web : `sudo apt-get install apache2`

# Préparation de MySQL

Pour que GLPI puisse fonctionner, il va falloir lui donner une base de données pour travailler. Si vous n'avez pas accès au serveur MySQL ou que vous n'êtes pas administrateur du serveur, il faudra vous assurer d'avoir les éléments suivants : l'adresse du serveur MySQL, votre identifiant MySQL, votre mot de passe MySQL et le nom de la base qui vous a été attribuée, ainsi que les droits dessus.

Rentrez dans MySQL :

```bash
mysql -u root -p
```

Et tapez les commandes suivantes :

```sql
mysql> CREATE DATABASE glpi;
mysql> CREATE USER 'glpi'@'localhost';
mysql> GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost' IDENTIFIED BY 'your_password';
mysql> FLUSH PRIVILEGES;
```

Pressez `CTRL-D` ou tapez `exit` pour sortir de MySQL. 

Voilà, votre base GLPI est prête.

# Mise en place de GLPI

## Récupération de l'archive

Pour récupérer la dernière version de GLPI, il suffit de se rendre sur la [page de téléchargement](http://www.glpi-project.org/?article3&lang=fr) et de prendre le lien de la dernière version stable.

```bash
wget https://github.com/glpi-project/glpi/releases/download/0.90.1/glpi-0.90.1.tar.gz
```

## Installation de l'archive

Décompressez l'archive téléchargée dans la racine de votre serveur Web (ici `/var/www/` pour Apache2) :

```bash
sudo tar xzf glpi-0.90.1.tar.gz -C /var/www
```

> Ce tutoriel a été fait pour la version 0.90.1 de GLPI, il faudra donc adapter la commande en fonction de la version téléchargée.

Donnez ensuite les droits à l'utilisateur qui gérera le serveur, ici `www-data` :

```
sudo chown -R www-data:www-data /var/www/glpi
```

Voilà, votre dossier GLPI est prêt. Il ne reste plus qu'à créer un serveur virtuel pour le serveur GLPI.

# Configuration d'Apache

GLPI possède l'avantage d'être configuré, dès le début, via son interface Web. Ce qui permet de ne rien avoir à configurer au préalable dans des dizaines de fichiers de configuration. Mais pour cela, il vous faut un serveur Web capable de le rendre accessible.

Créer un nouveau serveur virtuel (`sudo vi /etc/apache2/sites-available/glpi.conf`) et éditez-le comme suit :

```conf
<VirtualHost *:80>
        ServerName glpi.local.fr
        ServerAlias glpi

        DocumentRoot /var/www/glpi
        <Directory /var/www/glpi>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
                AuthType Basic
        </Directory>

        LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-agent}i\"" combined
        CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
        ErrorLog  ${APACHE_LOG_DIR}/glpi_error.log

</VirtualHost>
```

Enregistrez et quittez.

Activez le site et relancer Apache2 :

```bash
sudo a2ensite glpi.conf
sudo service apache2 reload
```

> Il peut arriver que le site par défaut d'Apache2 pose problème et vous emmène sur la page d'accueil d'Apache au lieu de celle de GLPI. Dans ce cas, désactivez-le (`a2dissite 000-default.conf`) et recharger la configuration d'Apache2.

# Configurer votre DNS

Si vous le pouvez entrez l'adresse de votre nouveau serveur Web dans votre DNS en lui associant un HOTE et un ALIAS si besoin. Si vous n'avez pas de DNS sous la main, éditez votre fichier Hosts qui se trouve dans `/etc/` comme suit :

```conf
ip_serveur glpi.local.fr
```

Remplacez *ip_serveur* par votre IP.

Enregistrez et quittez.

# Interface GLPI

## Vérifications des dépendances

Maintenant, si vous ouvrez votre navigateur sur l'adresse suivante : [http://glpi.local.fr](http://glpi.local.fr), vous devriez tomber sur l'interface de GLPI :

<figure>
    <img src="/images/glpi/glpi_accueil.png" alt="">
    <figcaption>GLPI - install.php</figcaption>
</figure>

Vous n'avez plus qu'à suivre les différentes étapes :

* Sélectionnez votre language et cliquez sur **OK**.
* Acceptez ensuite les Termes et Conditions et la Licence.
* Après GLPI vous demandera si c'est une nouvelle version ou une mise à jour. Cliquez sur **Installer**. (Comme vous l'avez peut-être deviné, les mises à jour de GLPI se feront quasiment de la même manière.)
* Enfin GLPI va vérifier votre environnement pour voir si tout est correct. Normalement, vous devriez avoir pas mal d'alertes signalées par des triangles rouges. Cela indique que GLPI n'a pas encorte tout ce qu'il veut. Notament certaines dépendances et certains droits.

Pour régler ce problème, exécutez les commandes suivantes :

```bash
sudo apt-get install php5-mysql php5-gd
sudo service apache2 restart
```

Cliquez ensuite sur le bouton **Réessayer**.

Si tout va bien, vous devriez avoir tous les voyants au vert :

<figure>
    <img src="/images/glpi/glpi_setup.png" alt="">
    <figcaption>GLPI Setup</figcaption>
</figure>

> Si GLPI vous dit que la commande est erronée, il suffit de **re-faire** pointer votre navigateur sur `http://glpi.local.fr/install.php` pour que GLPI relance l'installation.

Cliquez sur **Continuer**.

## Configuration de la base de données

Maitenant, GLPI va vous demander les données de la base MySQL. Si votre serveur MySQL est sur la même machine que GLPI, vous devrez mettre _localhost_ en face de Serveur MySQL. Voici à quoi cela devrait ressembler

<figure>
    <img src="/images/glpi/glpi_bdd.png" alt="">
    <figcaption>GLPI - Base de Données</figcaption>
</figure>

Cliquez sur **Continuer** puis au prochain écran sélectionnez la base que vous avez créée, ici _glpi_. Vous pouvez remarquer que GLPI vous propose aussi d'en créer une. C'est une autre possibilité, mais vu que vous avez déjà créé votre base autant la garder.

Enfin GLPI devrait vous dire que la base a bien été initialisée :

<figure>
    <img src="/images/glpi/glpi_bdd_ok.png" alt="">
    <figcaption>GLPI - Base de Données initialisée !</figcaption>
</figure>

Vous êtes arrivé à la fin de l'installation.

## Fin de l'installation

GLPI vous donne plusieurs informations à la fin de l'installation :

<figure>
    <img src="/images/glpi/glpi_install_finie.png" alt="">
    <figcaption>GLPI - Base de Données initialisée !</figcaption>
</figure>

Lisez les attentivement !

Entrez le login admin (Login : glpi / Mot de passe : glpi) lorsque GLPI vous le demande.

Vous êtes maintenant sur l'interface de GLPI. Vous devriez avoir deux avertissements :
 
* Plusieurs comptes par défaut, dont celui de l'administrateur GLPI, sont déjà présents. **Je vous conseille de les modifier dès votre première connexion** via l'interface.
* Le fichier install/install.php n'est plus utile et présente un risque pour votre installation. En effet, n'importe qui peut venir écraser votre installation en rejouant les étapes que vous venez de faire. **Supprimez-le !**

```bash
sudo rm install/install.php
```

Félicitations ! Votre serveur GLPI est prêt.

# Conclusion

GLPI est très facile à monter comme serveur et (dans de futurs tutoriels) vous verrez qu'il est capable de faire beaucoup de choses automatiquement. N'hésitez pas à fouiller dans le [catalogue des plugins](http://plugins.glpi-project.org/#/). Il est très fourni et plutôt bien fait : il permet de s'abonner aux différents plugins, de les rechercher par type et offre un lien (si possible) vers leurs dépôts.

De plus, la communauté GLPI est très sympathique et réactive, pour peu qu'on se donne la peine de faire des recherches, un minimum, avant de poser des questions.
