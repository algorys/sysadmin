---
layout: post
title: Install Dokuwiki
lang: en
ref: dokuwiki
modified:
description: How to install Dokuwiki
tags: [tutoriel, wiki]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2015-11-21T22:05:05+01:00
---

# Introduction

## Why Dokuwiki ?

If you do not know this kind of tool , visit [Wikipedia : Wiki](https://fr.wikipedia.org/wiki/Wiki) to learn a little more.

[DokuWiki](https://www.dokuwiki.org/) is simple to install and use. Furthermore it is possible to do many things with because developing your own plugins is fairly easy as long as you have some programming knowledge.

## To know

The interest of such a site is that it allows you to establish a full internal documentation, either for your business or other organization, game universe, etc ...

This tutorial will be compatible with any debian-like (Debian, Ubuntu, Mint...). Commands must be adapted if using other Linux distributions.

## Prerequisites

  * A Linux Server
  * A Web server [Apache]( http://www.apache.org/httpd) or [nginx](http://nginx.org/) for example.

```bash
sudo apt- get install apache2
```

You must have PHP installed :

```bash
sudo apt- get install php5
```

That's all.

# Download and Install

On some distributions, it is quite possible to install DokuWiki in one command (`sudo apt-get install dokuwiki`).

> **However, I advise you not to do that.**

The installation package installs Dokuwiki in several folders, which is not necessarily the best choice. As much put everything in the same file to facilitate future updates. You will see it is very simple and for that matter, as your install server wherever you want !

Visit the [Download page](http://download.dokuwiki.org/) Dokuwiki and copy the link to the version "Stable".

> Open a terminal and paste the copied link:

```bash
cd /var/www/
sudo wget http://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
```

Unzip the downloaded archive and rename it :

```bash
sudo tar xzvf dokuwiki-stable.tgz
sudo mv dokuwiki-DATE-VERSION monwiki
sudo rm dokuwiki-stable.tgz
```

Give the rights to web server user (`www-data` for Apache2) :

```bash
sudo chown -R www-data:www-data data conf
```

That's done, the folder for your server is ready.

# Configure Apache2

We will now tell to Apache to run it. If Apache is not installed, install it now (`sudo apt-get install apache2`).

> Add a configuration file for your Wiki into folder of your web server.

```conf
<VirtualHost *:80>
    # Détails du serveur
    ServerName monwiki.domaine.com
    ServerAlias monalias

    ServerAdmin admin@mail.com

    # Options
    DocumentRoot /var/www/monwiki
    <Directory /var/www/monwiki>
    Options Indexes FollowSymLinks MultiViews
      Order allow,deny
      allow from all
    </Directory>

    # Logs
    ErrorLog ${APACHE_LOG_DIR}/wiki_error.log
    CustomLog ${APACHE_LOG_DIR}/wiki_access.log combined

</VirtualHost>
```

> Activate the site and reload your web server :

```conf
sudo a2ensite monwiki.conf
sudo service apache2 reload
```

Open your browser and enter the following address by replacing `ip_server` by your ip :

```php
http://ip_server/install.php
```

You should see the installation page of Dokuwiki.

If you have a blank page, or you have an error page, read again this tutorial. Otherwise check the Apache logs.

## Web interface

Now that you have an operational server, you must customize it.

Fill out the fields :

 * Enter a name for your wiki.
 * Choose a root user and password (this is the login of the wiki administrator, but you can add more later)
 * Make a choice between different wiki policies.

And click on **Save**.

If all goes well, you should arrive on the homepage of your wiki. You can log-in as root you've previously created.

# Conclusion

Mount a wiki server with Dokuwiki is extremely simple. In addition there are many [plugins](https://www.dokuwiki.org/plugins) and many [themes](https://www.dokuwiki.org/template) to personalize it.

