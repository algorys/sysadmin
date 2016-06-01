---
layout: post
title: Install GLPI
lang: en
ref: glpi
modified:
description: How to install glpi
tags: [tutoriel, glpi]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-03-07T16:52:05+01:00
---

# Introduction

[GLPI](http://www.glpi-project.org/) is an IT manager for managing the resources of your IT Infrastructure, either in hardware, software or network. It also allows, through its various plugins to suit virtually any type of computer equipment. Moreover GLPI is very easy to install, it can interact with other servers to automatically collect its data ([OCS](http://www.ocsinventory-ng.org/fr/), [Fusion-Inventory](http://fusioninventory.org/)) and may even do the monitoring by coupling with a server [Shinken](http://www.shinken-monitoring.org/) (or[Nagios](https://www.nagios.org/)).

# Prerequisites

To install GLPI, you need to have the following installed :

* PHP 5.3 or more : `sudo apt-get install php5`
* A MySQL database : `sudo apt-get install mysql-server`
* A webserver : `sudo apt-get install apache2`

That's all.

# Prepare MySQL

First, it will have to give it a database to work. If you do not have access to MySQL server or you are not an administrator of the server, it will ensure you have the following: the IP address of the MySQL server, the MySQL username, your MySQL password and database name that was assigned to you, and obviously the rights for.

Go into MySQL:

```bash
mysql -u root -p
```

And type the following commands :

```sql
mysql> CREATE DATABASE glpi;
mysql> CREATE USER 'glpi'@'localhost';
mysql> GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost' IDENTIFIED BY 'your_password';
mysql> FLUSH PRIVILEGES;
```

Press `CTRL-D` or type `exit` to exit of MySQL.

Your database is ready for GLPI.

# GLPI

## Get Archive

To retrieve the latest version of GLPI, simply go on [download page](http://www.glpi-project.org/spip.php?article41) and take the link to the latest stable version.

```
wget https://github.com/glpi-project/glpi/releases/download/0.90.3/glpi-0.90.3.tar.gz
```

## Install Archive

Unzip the downloaded archive into the root of your Web server (here `/var/www/` for Apache2):

```bash
sudo tar xzf glpi-0.90.1.tar.gz -C /var/www
```

> This tutorial was made for GLPI 0.90.X, so adapt commands according to your version.

Next, give rights to user of webserver, here `www-data`:

```
sudo chown -R www-data:www-data /var/www/glpi
```

Now folder of GLPI is ready. It remains only to create a virtual server for the GLPI server.

# Apache Configuration

GLPI has the advantage of being set from the beginning, through its Web interface. This allows to have nothing to configure in advance in dozens of configuration files. But for this you need a Web server that make it accessible.

Create a new virtual server (`sudo vi /etc/apache2/sites-available/glpi.conf`) and edit as follow :

```conf
<VirtualHost *:80>
        ServerName glpi.local.en
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

Save and exit.

Activate site and relaunch Apache:

```bash
sudo a2ensite glpi.conf
sudo service apache2 reload
```

> Sometimes the apache2 default website interferes and takes you on the Apache home page instead of the GLPI. In this case, disable it (`a2dissite 000-default.conf`) and reload the configuration of Apache2.

# Configure your DNS

If you can, enter IP adress of you server in your DNS by associating an HOSTNAME and an ALIAS if needed. If not, edit your Hosts file that is in `/etc/` like that :

```conf
ip_server glpi.local.en
```

Replace *ip_sever* by your IP.

Save and quit.

# GLPI user Interface

## Check Dependencies

Now, if you open your browser on : [http://glpi.local.en](http://glpi.local.en), you must have :

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_accueil.png" alt="">
    <figcaption>GLPI - install.php</figcaption>
</figure>

You just have to follow the steps:

* Select your language and click **OK**.
* Then accept the Terms and Conditions and the Licence.
* After GLPI ask if this is a new version or an update. Click **Install**. (As you may have guessed, GLPI updates the same way.)
* Finally GLPI will check your environment to see if everything is correct. Normally, you should have a lot of alerts reported by red triangles. This indicates that GLPI has not all he wants. Including some dependencies and certain rights.

To correct this problem , try to run the following commands:

```bash
sudo apt-get install php5-mysql php5-gd
sudo service apache2 restart
```

Then click on **Retry** button. If all goes well, you you should have all the lights green :

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_setup.png" alt="">
    <figcaption>GLPI Setup</figcaption>
</figure>

> If GLPI says that command is wrong, just go again with your browser to root url `http://glpi.local.en/install.php` to relaunch install.

Click on **Continue**.

## Configure Database

Now, GLPI will ask you data of your MySQL database. If your MySQL server is on the same server, you should put _localhost_ in MySQL Server field. Here is what it should look like:

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_bdd.png" alt="">
    <figcaption>GLPI - Base de Données</figcaption>
</figure>

Click **Continue** and then the next screen select the database you created here _glpi_. You may notice that GLPI also offers to create one. This is another possibility, but since you have already created your base as that you keep.

Finally GLPI should tell you that the base has been initialized:

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_bdd_ok.png" alt="">
    <figcaption>GLPI - Base de Données initialisée !</figcaption>
</figure>

You arrived at the end of the installation.

## Finalize install

GLPI gives you more information at the end of the installation:

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_install_finie.png" alt="">
    <figcaption>GLPI - Base de Données initialisée !</figcaption>
</figure>

Read carefully!

Enter the admin login (login : glpi / Password: glpi ) when GLPI prompted.

You are now on the GLPI interface. You should have two warnings :

* Multiple default accounts, including that of GLPI administrator, are already present. **I advise you to change from your first connection via the interface**.
* The file install/install.php is no longer useful and is a risk to your system. Indeed, anyone can come crashing your system by replaying the steps you just did. **Delete it !**

```bash
sudo rm install/install.php
```

Félicitations ! Votre serveur GLPI est prêt.

Congratulations ! GLPI your server is ready .

# Conclusion

GLPI is very easy to set up as a server and (in future tutorials) you will see that it is capable of doing many things automatically. Feel free to rummage through the [plugins catalog](http://plugins.glpi-project.org/#/). It is very dense and rather well done : it allows to subscribe to the various plugins, search by type, and link (if possible) to their repositories.

In addition, the GLPI community is very friendly and responsive, if only one takes time to research a minimum, before asking questions.
