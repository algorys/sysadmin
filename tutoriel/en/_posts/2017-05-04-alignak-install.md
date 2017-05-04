---
layout: post
title: Install Alignak
lang: en
ref: alignak
modified:
description: How to install Alignak Solution
tags: [tutoriel, alignak, monitoring]
image:
  feature:
  credit:
  creditlink:
author: algorys mohierf ddurieux
comments: true
share:
date: 2017-05-04T13:20:05+01:00
---

# Introduction

Alignak is a monitoring framework. It is used to check your IT (or over) and to keep you informed about problems and performance degradation (mail, SMS, XMPP…). It can be used from small environments to very large (multi-datacenter, fail-over, load-balancing).

**WARNING:** Alignak is still under development and may be unstable ! The first release is however very close.

During this tutorial, we'll install Alignak and its satellites, who are written mainly in Python. Those applications require a minimum knowledge with Linux and to be resourceful.

It can come in addition to the demo tutorial provided by the Alignak team: [Alignak Demo](https://github.com/Alignak-monitoring-contrib/alignak-demo).

Almost all of Alignak's repositories have stable versions available via `pip`. But I'll try to provide each repository URL each time you install a new software.

# Prerequisites

> **Note:** This tutorial was performed on an Ubuntu 16.04 LTS server. Other Linux are also compatible.

## Mandatory Requirements

First of all, like all installation, you need to have a server up-to-date:

```bash
sudo apt-get update && sudo apt-get upgrade
```

Then some tools are needed and obviously Python. Main satellites of Alignak are only compatible with Python 2.7 ! So if you have Python 3 installed, be sure `python` command is linked to Python 2.7.

Let's install:

```bash
sudo apt-get install git 
sudo apt-get install python2.7 python2.7-dev python-pip

# Needed for the PyOpenSSL / Cryptography dependencies of Alignak
sudo apt-get install libffi-dev libssl-dev
```

It is recommended also to upgrade default pip:

```bash
sudo pip install --upgrade pip
```

## Optional Requirements

This tutorial currently use **screen** because the scripts provided by Alignak use it:

```bash
sudo apt-get install screen
```

Here is some screen hint and tips to help you later:

```bash
# Listing the active screens
screen -ls

# Joining a screen
screen -r alignak-backend

# Leaving a screen (without killing it)
screen -r alignak-backend
Ctrl a+d

# Switching between active screens
Ctrl a+n
```

# Alignak Base

> **Note:** You've to know that all Alignak components need a root account (or sudo privileges) to get installed.

Alignak need also an **alignak** user. Don't create it yet, we'll create after with a script.

Many components will be installed with pip, only Framework is installed with Git. If you want to install from the repository also, it is recommended to use only **develop branch**. We try to make sure that the develop branch is always as stable as possible !

## Alignak Framework

> **Repos:** [Alignak](https://github.com/Alignak-monitoring/alignak)

Download the repository and install it:

```bash
mkdir ~/repos; cd ~/repos

# Alignak framework
git clone https://github.com/Alignak-monitoring/alignak
cd alignak/
# Install alignak and all its python dependencies
# -v will activate the verbose mode of pip
sudo pip install -v .
```

Pay attention to the end message,  it will already give you a lot of indications on what is good or not.

As you can see, alignak does not found **alignak** user / group and say also that you've to set some permissions on specific folders.

To correct this, run the following script:

```bash
sudo ./dev/set_permissions.sh
```

You'll have the following output:

```bash
Checking / creating 'alignak' user and users group
Checking / creating 'nagios' users group
Adding group `nagios' (GID 117) ...
Done.
Adding user 'alignak' to the nagios users group
Adding user `alignak' to group `nagios' ...
Adding user alignak to group nagios
Done.
Setting 'alignak' ownership on: /usr/local/etc/alignak
Setting 'alignak' ownership on: /usr/local/var/run/alignak
Setting 'alignak' ownership on: /usr/local/var/log/alignak
Setting 'alignak' ownership on: /usr/local/var/lib/alignak
Setting 'alignak' ownership on: /usr/local/var/libexec/alignak
Setting file permissions on: /usr/local/etc/alignak
Terminated
```

That's all for the framework.

## Alignak Backend

> **Repos:** [Alignak Backend](https://github.com/Alignak-monitoring-contrib/alignak-backend)

The backend of Alignak is the heart of solution. To run it, you'll need to have a running Mongo database (see above).

To install alignak-backend:

```bash
sudo pip install alignak-backend
```

You can also install the **alignak-backend-import** tool. Alignak-backend-import will be used to import your config files after.

> **Repos:** [Alignak Backend Import](https://github.com/Alignak-monitoring-contrib/alignak-backend-import)

```bash
sudo pip install alignak-backend-import
```

### Mongo

To install MongoDB, you can find more informations on [MongoDB Documentation](https://docs.mongodb.com/manual/).

But here is how to do that on Ubuntu 16.04 Xenial:

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo service mongod start
```

## Alignak Webui

> **Repos:** [Alignak WebUI](https://github.com/Alignak-monitoring-contrib/alignak-webui)

The WebUI is the the web interface for Alignak monitoring framework. To install it, run the followng command:

```bash
sudo pip install alignak-webui
```

In fact, if you want to use alignak you're yet ready. But alignak still has many components that are useful.

# Checks

> **Note:** You do not have to **install all the checks** below. Take what you are interested in and match your infrastructure.

This commands will be used to check the status of your services and each have a syntax of their own !

The most known is [Nagios Plugins](https://www.nagios.org/projects/nagios-plugins/). You can find a tutorial on this website on how install and use them and also for SNMP plugins:

* [Install Nagios and SNMP plugins](/2016/12/install-nagios-and-snmp-plugins)

There are many other checks available for each protocol on Alignak organization:

```bash
# Checks hosts thanks to NRPE Nagios active checks protocol
sudo pip install alignak-checks-nrpe

# Checks hosts thanks to old plain SNMP protocol
sudo pip install alignak-checks-snmp

# Checks hosts with "open source" Nagios plugins (eg. check_http, check_tcp, ...)
sudo pip install alignak-checks-monitoring

# Checks mysql database server
sudo pip install alignak-checks-mysql

# Checks Windows passively checked hosts/services (NSClient++ agent)
# As of now, use ==1.0rc1 to get the correct version
sudo pip install alignak-checks-windows-nsca

# Checks Windows with Microsoft Windows Management Instrumentation
sudo pip install alignak-checks-wmi
```

You can add later if necessary.

# Alignak Modules

Alignak modules add some features to Alignak Framework. Some are necessary for the essential alignak features (like retention or logs), others are fully optionnal.

## Backend Module

> **Repos:** [Alignak Module Backend](https://github.com/Alignak-monitoring-contrib/alignak-module-backend)

The Alignak module Backend is needed for retention, logs, timeseries, ...

```bash
sudo pip install alignak-module-backend
```

## Logs Module

> **Repos:** [Alignak Logs Backend](https://github.com/Alignak-monitoring-contrib/alignak-module-logs)

The logs module add some logs for all monitoring events: alerts, notifications... This module is very usefull.

```bash
sudo pip install alignak-module-logs
```

## Notifications

> **Repos:** [Alignak Notifications](https://github.com/Alignak-monitoring-contrib/alignak-notifications)

This package will add some command to send notifications via email, XMPP, ... This is usefull if you want to be prevent when something goes wrong or not:

```bash
sudo pip install alignak-notifications
```

## Other Modules

> **Repos:** The following modules are all available under [Alignak Contrib](https://github.com/Alignak-monitoring-contrib/) organization.

These modules add other commands or services.

```bash
# Collect passive NSCA checks
sudo pip install alignak-module-nsca
# Write external commands (Nagios-like) to a local named file
sudo pip install alignak-module-external-commands
# Notify external commands though a WS and get Alignak state with your web browser
sudo pip install alignak-module-ws
# Improve NRPE checks
sudo pip install alignak-module-nrpe-booster
```

# Configure Alignak

Now you normally have everything you need to get started.

As you will see, most of the configuration will be located in `/usr/local/etc`. Inside this folder, you'll find the _alignak_, _alignak-backend_ and _alignak-webui_ folders.

Other will install there configuration files inside `/usr/local/etc/alignak` (like alignak-notification for example). With this system alignak developers want the configuration of Alignak and its satellites to be centralized maximum.

> **Note:** All alignak configuration files are widely commented and detailed for each options ! Read carefully before making any changes.

## Check Installation

Perform these checks before.

For Python packages:

```bash
# The main packages you must have:
pip list --format=columns | grep alignak
alignak                0.2        
alignak-backend        0.8.17     
alignak-backend-client 0.9.0      
alignak-backend-import 0.9.0      
alignak-module-backend 0.5.0      
alignak-module-logs    0.5.1      
alignak-notifications  0.3.1      
alignak-webui          0.8.3.4 
```

For alignak folders:

```bash
ls -la /usr/local/etc/
total 20
drwxr-xr-x  5 root    root    4096 May  4 07:04 .
drwxr-xr-x 11 root    root    4096 May  4 06:44 ..
drwxrwxr-x  6 alignak alignak 4096 May  4 08:01 alignak
drwxr-xr-x  2 root    root    4096 May  4 07:04 alignak-backend
drwxr-xr-x  2 root    root    4096 May  4 07:04 alignak-webui

ls -la /usr/local/etc/alignak
total 44
drwxrwxr-x 6 alignak alignak 4096 May  4 08:01 .
drwxr-xr-x 5 root    root    4096 May  4 07:04 ..
-rwxr-xr-x 1 root    root    6445 Jan 29 21:20 alignak.backend-run.cfg
-rw-rw-r-- 1 alignak alignak 7756 May  4 06:44 alignak.cfg
-rw-rw-r-- 1 alignak alignak 3808 May  4 06:44 alignak.ini
drwxrwxr-x 8 alignak alignak 4096 May  4 06:44 arbiter
drwxrwxr-x 2 alignak alignak 4096 May  4 06:44 certs
drwxrwxr-x 2 alignak alignak 4096 May  4 06:44 daemons
drwxrwxr-x 3 alignak alignak 4096 May  4 06:44 sample

ls -la /usr/local/var/log/
total 20
drwxr-xr-x 5 root    root    4096 May  4 07:04 .
drwxr-xr-x 6 root    root    4096 May  4 06:44 ..
drwxr-xr-x 2 alignak alignak 4096 May  4 06:44 alignak
drwxr-xr-x 2 root    root    4096 May  4 07:04 alignak-backend
drwxr-xr-x 2 root    root    4096 May  4 07:04 alignak-webui
```

As you can see some files in alignak folder miss _alignak_ user / group and the same thing for **log** folder.

To solve that, run the following commands:

```bash
cd ~/repos/alignak
sudo ./dev/set_permissions.sh
sudo chown alignak:alignak /usr/local/var/log/alignak*
```

Now your installation is `OK` !

We'll begin by Backend and WebUI folder who are the more simple.

## Configure Backend

You will find 2 files in backend folder:

* uwsgi.ini
* settings.json

Right now, just edit the following file:

```bash
sudo vi /usr/local/etc/alignak-backend/uwsgi.ini
```

In this file you can see that backend is by default on `0.0.0.0:5000`. You can change IP and Port but don't forget after to report this change to other files !

The JSON file contains others configurations essentially for Bakcend data, Mongo or Timeseries.

## Configure WebUI

In WebUI Folder, you'll find 3 files:

* uwsgi.ini
* settings.cfg
* logging.json

Open the `uwsgi.ini` file. You will see that this file looks much like the backend. WebUI is on `0.0.0.0:5001` by default. Personnaly I prefer to use port 80, so:

```conf
http-socket = 0.0.0.0:80
```

Then open `settings.cfg` the WebUI configuration file. If you've modify IP and port of your WebUI, report your chnage here. The same for backend:

```ini
[bottle]
host = 0.0.0.0
port = 80

; If you've change backend IP:PORT, search and uncomment alignak_backend line
; alignak_backend = http://127.0.0.1:5000

; If you installed Web Service module, do the same.
; alignak_backend = http://127.0.0.1:5000
```

There is many options available to customize your WebUI.

The last file if for log formatting. You can have a look if you want, but it is not necessary to modify it.

## Configure Framework


