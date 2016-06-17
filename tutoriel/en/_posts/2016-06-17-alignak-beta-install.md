---
layout: post
title: Install Beta Alignak
lang: en
ref: alignak-beta
modified:
description: How to install Alignak Suite
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

# WIP : Tutorial in progress

# Introduction

Alignak is a monitoring solution. This solution is used to check your IT (or over) and to be informed about problems and performances degradation (mail, SMS, XMPP…). It can be used from small environments to very large (multi-datacenter, fail-over, load-balancing).

**WARNING:** Alignak is still under development and may be unstable ! This tutorial is only for those who want to test this server.

During this tutorial, we'll install the following servers:

* Alignak: daemons of Alignak.
* Alignak-Backend: backend of Alignak.
* Alignak-Webui: WebUI for Alignak Backend.

These servers require a minimum knowledge in Linux and be resourceful. 

Following installations are made with **git** (to get the last fixed) but for each you can install with `pip` too ! Just make for each step instead of `git`:

* `pip install alignak`
* `pip install alignak-backend`
* `pip install alignak-webui`

In this case, steps of installations are not yet required (like `sudo python setup.py install`).

**Be carefull, do not mix installations type !**

Now that you warned, you can start to play ;) !

# Prerequisites

For all you need to install the following dependencies:

* Server up to date: `sudo apt-get update && sudo apt-get upgrade`
* Python (**2.7** is recommended) and libs: `sudo apt-get install python2.7 python2.7-dev python-pip`
* Git: `sudo apt-get install git`

Check your version of Python:

```bash
$ python --version
# Output:
Python 2.7.6
```

**Tips:** You can upgrade `setuptools`, `pip` and `pbr` if you encounter problems with `pip install`.

You have to create a user _alignak_ :

`sudo adduser alignak`

# Alignak Daemons

First we need to get sources of Alignak:

```bash
cd ~; mkdir repos; cd repos
git clone https://github.com/Alignak-monitoring/alignak.git
cd alignak
```

Install dependencies with _pip_ and run _setup.py_:

```bash
sudo pip install -r requirements.txt
sudo python setup.py install
```

Normally you'll have something ending like:

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

If you don't have create _alignak_ user, **do it now** !

You must give rights to certain folders:

```bash
sudo chown -R alignak:alignak /usr/local/var/run/alignak
sudo chown -R alignak:alignak /usr/local/var/log/alignak
```

Now connect with _alignak_ and run **alignak** daemons:

```bash
sudo su - alignak
/usr/local/etc/init.d/alignak start
```

Normally, the following should appears in your terminal:

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

That's done, alignak daemons are started !

# Alignak Backend

## Prerequisites of Backend

You have to install `MongoDB` and `uwsgi` to run _alignak-backend_, so here we go:

```bash
sudo apt-get install mongodb uwsgi uwsgi-plugin-python
```

## Get the sources

Now you have alignak daemons running, you must get alignak backend running on your server. Go back to your user (`CTRL-D`) and clone again:

```bash
cd ~/repos
git clone https://github.com/Alignak-monitoring-contrib/alignak-backend.git
cd alignak-backend
```

Then install requirements with _pip_ and launch _setup.py_

```bash
sudo pip install -r requirements.txt
sudo python setup.py install
```

## Launching alignak-backend

Now you have install alignak-backend, we must create a folder for starting our app:

```bash
cd ~; mkdir -p app/alignak-backend; cd app/alignak-backend
# create symlink, like this update when updating repos
ln -s ~/repos/alignak-backend/alignak_backend.py ~/app/alignak-backend/alignakbackend.py
```

Then launch app like this:

```bash
# Replace xxx.xxx.xxx.xxx by IP of your server
sudo uwsgi --wsgi-file alignakbackend.py --callable app --socket xxx.xxx.xxx.xxx:5000 --protocol=http --enable-threads
```

> **Fix and Tips:** If you encounter difficulties, please check you have _MongoDB_ and _uwsgi-plugin-python_ installed. Check error during process. Try to remove any _*.pyc_ in current folder. Try logout and login to see if that's not a problem of updating paths.

# Alignak WebUI

Now we have alignak daemons started by user alignak and alignak-backend starting by sudo and uwsgi. We can install alignak-webui ! Let's go:

```bash
cd ~/repos
git clone https://github.com/Alignak-monitoring-contrib/alignak-webui.git
cd alignak-webui
```

You an now proceed to install:

```bash
sudo pip install -r requirements.txt
sudo python setup.py install
```

Then go back to our folder for apps and create new symlink:

```bash
cd ~/app; mkdir alignak-webui; cd alignak-webui
ln -s ~/repos/alignak-webui/alignak_webui/app.py ~/app/alignak-webui/app.py
# TODO: to test again
# cp ~/repos/alignak-webui/etc/settings.cfg settings.cfg
```

You can set several configuration in _settings.cfg_.

Now you can get some information about _app.py_. Just run `sudo python app.py -h` for more help. And now go to launch it:

```bash
# Replace xxx.xxx.xxx.xxx by IP of your server or your FQDN.
sudo python app.py -n xxx.xxx.xxx.xxx -p 80 &
# In case of it doesn't take right backend but default
sudo python app.py -b http://xxx.xxx.xxx.xxx:5000 -n xxx.xxx.xxx.xxx -p 80 &
```

Now your WebUI has started and you can reach : http://xxx.xxx.xxx.xxx on your browser. You can have some logs in your current folder:

```bash
sudo tail -f ~/app/alignak-webui/alignak-webui.log
```

# WIP
