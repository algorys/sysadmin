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

Alignak is a monitoring framework. It is used to check your IT (or over) and to keep you informed about problems and performance degradation (mail, SMS, XMPP…). It can be used from small environments to very large (multi-datacenter, fail-over, load-balancing).

**WARNING:** Alignak is still under development and may be unstable ! This tutorial is only for the one who want to test this application.

During this tutorial, we'll install the following applications:

* Alignak: daemons of Alignak.
* Alignak-Backend: backend (database) of Alignak.
* Alignak-Webui: WebUI for Alignak Backend.

Those applications require a minimum knowledge with Linux and to be resourceful.

Following installations are made with **git** (to get the last fixed) but for each you can install with `pip` too ! Just make for each step instead of `git`:

* `pip install alignak`
* `pip install alignak-backend`
* `pip install alignak-webui`

In this case, steps of installations are not yet required (like `sudo python setup.py install`).

**Be carefull, do not mix installations type !**

Now that you aware, you can start to play ;) !

# Prerequisites

For all the applications, you need to install the following dependencies:

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

You have to create a user _alignak_ and connect as this user:

```bash
sudo adduser alignak
# Here I give sudo right to alignak to facilitate installation.
# You can switch with a sudo user instead.
sudo adduser alignak sudo
sudo su - alignak
```

# Alignak Daemons

First we need to get sources of Alignak. We try to keep it ordered, so let's create a folder call _repos_ before:

```bash
cd ~; mkdir repos; cd repos
git clone https://github.com/Alignak-monitoring/alignak.git
cd alignak
```

Install dependencies with _pip_ and run _setup.py_:

```bash
# No need of root for "pip"
pip install -r requirements.txt
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

You must give rights to certain folders (this because setup.py run as root):

```bash
sudo chown -R alignak:alignak /usr/local/var/run/alignak
sudo chown -R alignak:alignak /usr/local/var/log/alignak
sudo chown -R alignak:alignak /usr/local/etc/alignak
```

Now ensure you're logged-in as _alignak_ and run **alignak** daemons:

```bash
/usr/local/etc/init.d/alignak start
```

Normally, the following should appear in your terminal:

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

Now you have alignak daemons running, you must get alignak backend running on your server. Stay logged-in as user _alignak_ and type:

```bash
cd ~/repos
# keep repos in backend
git clone https://github.com/Alignak-monitoring-contrib/alignak-backend.git backend
cd backend
```

Then install requirements with _pip_ and launch _setup.py_

```bash
pip install -r requirements.txt
sudo python setup.py install
```

## Launching alignak-backend

Now you have installed alignak-backend, we must create a folder for starting our app. But not in the _repos_ folder but in a new one called **app**:

```bash
cd ~; mkdir -p app/backend; cd app/backend
# create symlink, like this update when updating repos
ln -s ~/repos/backend/alignak_backend.py ~/app/backend/alignakbackend.py
```

Then launch app like this:

```bash
# Replace xxx.xxx.xxx.xxx by IP of your server
uwsgi --plugin python --wsgi-file alignakbackend.py --callable app --socket xxx.xxx.xxx.xxx:5000 --protocol=http --enable-threads
```

> **Fix and Tips:** If you encounter difficulties, please check you have _MongoDB_ and _uwsgi-plugin-python_ installed. Check error during process. Try to remove any _*.pyc_ in current folder. Try logout and login to see if that's not a problem of paths updating.

# Alignak Backend modules

Now you have alignak daemons and backend running, but both do not yet communicate each other ... you need to install and configure some Alignak daemons modules.

## Get the sources

Stay logged-in as user _alignak_ and type:

```bash
cd ~/repos
# keep repos in backend
git clone https://github.com/Alignak-monitoring-contrib/alignak-module-backend.git backend-modules
cd backend-modules
```

Then install requirements with _pip_ and launch _setup.py_

```bash
pip install -r requirements.txt
sudo python setup.py install
```

## Configuring Alignak modules

The installation setup script created some configuration files in your */usr/local/etc/alignak/arbiter_cfg/modules* directory.

You must configure the backend URL and login information in the three following files:

```bash
cd /usr/local/etc/alignak/arbiter_cfg
sudo vi modules/mod-alignakbackendarbit.cfg
sudo vi modules/mod-alignakbackendarbit.cfg
sudo vi modules/mod-alignakbackendarbit.cfg
```

> **Hint:** Currently, you simply have to uncomment the `username` and `password` variables to allow connection to the backend.

Then you need to configure Alignak daemons to inform about the existing modules:
```bash
cd /usr/local/etc/alignak/arbiter_cfg
sudo vi daemons_cfg/arbiter-master.cfg

    # Backend module
    modules    	 alignakbackendarbit


sudo vi daemons_cfg/broker-master.cfg

    # Backend module
    modules    	 alignakbackendbrok


sudo vi daemons_cfg/scheduler-master.cfg

    # Backend module
    modules    	 alignakbackendsched

```

> **Note:** Currently, the scheduler backend module is broken and you should not configure it! Fixes is coming soon for the data retention...


Once you configured Alignak daemons, restart Alignak and that's all:
```bash
/usr/local/etc/init.d/alignak restart
```

> **Tips:** If an error occured, you will have some information about it in the log files located in */usr/local/var/log/alignak*.


# Alignak Backend import tool

Now you have alignak daemons and backend running, you must fill the backend database with some data about the hosts and services you need to monitor. The Alignak backend provides a REST API that you can use with cUrl or any other tool like Postman, but what about importing your Nagios-like flat files configuration auto-magically ?

## Get the sources

Stay logged-in as user _alignak_ and type:

```bash
cd ~/repos
# keep repos in backend
git clone https://github.com/Alignak-monitoring-contrib/alignak-backend-import.git backend-import
cd backend-import
```

Then install requirements with _pip_ and launch _setup.py_

```bash
pip install -r requirements.txt
sudo python setup.py install
```

## Launching alignak-backend-import

The alignak-backend-import setup script creates a script called `alignak_backend_import` in your */usr/local/bin* directory.

You can run the installed command line script with some options. Try this to get the available command line options:

```bash
alignak_backend_import -h
```

An exemple configuration is available in the project directory: *test/shinken_cfg_files/default*. You can import this configuration in your Alignak backend with this command:

```bash
alignak_backend_import -d -b http://127.0.0.1:5000 test/shinken_cfg_files/default/_main.cfg
```

Some explanations:

- `-d` delete the former existing elements in the backend
- `-b http://127.0.0.1:5000` specifies to use the local backend on port 5000
- `test ... _main.cfg` is the flat files configuration entry point


> **Fix and Tips:** A detailed documentation is available here: http://alignak-backend-import.readthedocs.io/en/latest/index.html.


Restart Alignak to make it aware of your new configuration:
```bash
/usr/local/etc/init.d/alignak restart
```

# Alignak WebUI

Now we have alignak daemons and alignak-backend started by the user _alignak_, we can install alignak-webui ! Let's go:

```bash
cd ~/repos
git clone https://github.com/Alignak-monitoring-contrib/alignak-webui.git webui
cd webui
```

You an now proceed to install:

```bash
pip install -r requirements.txt
sudo python setup.py install
# Make run.sh executable
chmod +x run.sh
```

Then go back to our folder for apps and create a new symlink:

```bash
cd ~/app; mkdir webui; cd webui
ln -s ~/repos/webui/run.sh ~/app/webui/run.sh
cp ~/repos/webui/alignak_webui.py alignakwebui.py
cp ~/repos/webui/etc/settings.cfg settings.cfg
```

The main _settings.cfg_ file is located in */etc/alignak_webui* (or */usr/local/etc/alignak-webui*). The application will read this main file and overload its variables with the one found in the *./settings.cfg* and *./etc/settings.cfg* files if they exist.

You can set several configuration in _settings.cfg_ but you must set your backend URL:

```conf
# settings.cfg
alignak_backend = http://127.0.01:5000
```

Then launch app like this:

```bash
# Use 0.0.0.0 to listen on all interfaces of your server, else you can specify 127.0.0.1
uwsgi --plugin python --wsgi-file alignakbackend.py --callable app --socket 0.0.0.0:8868 --protocol=http --enable-threads
```

It also exists a ``run.sh`` script that include this command line. If you do not start this script from the application directory, you must give absolute path for the *alignak_webui.py* file. Otherwise you can encouter this error:

```bash
--- no python application found, check your startup logs for errors ---
```

Edit this file as follow:

```conf
uwsgi \
--plugin python \
--wsgi-file /home/alignak/app/webui/alignakwebui.py \
--callable app \
--socket xxx.xxx.xxx.xxx:8868 \
--protocol=http \
--enable-threads
```

And run it:

```bash
./run.sh
```

Now your WebUI has started and you can reach : http://xxx.xxx.xxx.xxx:8668 on your browser. You can log in with the default following credentials:

* username: admin
* password: admin

The application builds a log file. The configuration for this log is defined in the _settings.cfg_ file, `logs` section. If the user rights permit, the log file is (as default) stored in the */var/log/alignak-webui* (or */usr/local/var/log/alignak-webui*) directory; else, the log can be tailed in the current folder:

```bash
tail -f ~/app/alignak-webui/alignak-webui.log
```

# WIP
