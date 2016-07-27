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
author: algorys mohierf
comments: true
share:
date: 2016-06-17T13:20:05+01:00
---

# WIP : Tutorial in progress (almost finished ;))

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

# Prerequisites

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
# Here you give sudo right to alignak to facilitate installation.
# You can switch with a sudo user instead.
sudo adduser alignak sudo
sudo su - alignak
```

# Alignak

## Alignak installation and start

First we need to get sources of Alignak. We try to keep it ordered, so let's create a folder call _repos_ before:

```bash
cd ~; mkdir repos; cd repos
git clone https://github.com/Alignak-monitoring/alignak.git
cd alignak
```

Install dependencies with _pip_ and run _setup.py_:

```bash
# No need of root for "pip". If you encounter problems with install, please retry with '--user' flag.
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

If you had not yet created _alignak_ user, **do it now** !

You must give rights to certain folders (this because setup.py runs as root):

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

## Alignak configuration

Alignak configuration files are located in the */usr/local/etc/alignak* directory. Main directories:

    - certs
    - daemons: daemons base configuration (communication, directories, ...)
    - arbiter_cfg:
      + daemons_cfg: configuration (modules, ...)
      + modules: modules configuration files
      + resource.d: resources configuration
      + objects: Nagios-like flat files configuration for monitored objects

## Nagios Plugins

To launch checks on host/services, Alignak need to have some installed plugins. Most often used plugins are the Nagios one. To install Nagios plugins, it's simple, you've just to download it on official website : [Nagios-plugins](https://nagios-plugins.org/).

```bash
cd ~; mkdir tools; cd tools
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
tar -xzvf nagios-plugins-2.1.1.tar.gz
# Now install it
cd nagios-plugins-2.1.1/
./configure --with-nagios-user=alignak --with-nagios-group=alignak
make
sudo make install
```

Your plugins are installed. They may not be installed in the directory `/usr/lib/nagios/plugins`. In this case , search for where they are (`whereis nagios` for example) and edit the file `/usr/local/etc/alignak/arbiter_cfg/resource.d/paths.cfg` accordingly:

```bash
vi /usr/local/etc/alignak/arbiter_cfg/resource.d/paths.cfg

    $NAGIOSPLUGINSDIR$=/usr/local/nagios/libexec
```

Once installation finished, restart Alignak to make it know the new parameters:

```bash
/usr/local/etc/init.d/alignak restart
```

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
# If you encounter problems with pip install, please retry with '--user' flag.
pip install -r requirements.txt
sudo python setup.py install
```

## Launching alignak-backend

Now that you have installed alignak-backend, you must start the backend.

The Alignak backend is a Python WSGI compliant application. As of it, it is very efficient to use uWSGI as a launcher to allow concurrency, monitoring, ...

It exists several solutions:

### Create your own application starter

Now that you have installed alignak-backend, you must create a folder for starting our app. But not in the _repos_ folder but in a new one called **app**:

```bash
cd ~; mkdir -p app/backend; cd app/backend
# create symlink, like this update when updating repos
ln -s ~/repos/backend/bin/alignak_backend.py ~/app/backend/alignakbackend.py
```

Then launch the application like this:

```bash
# Replace xxx.xxx.xxx.xxx by IP of your server
uwsgi --plugin python --wsgi-file alignakbackend.py --callable app --socket xxx.xxx.xxx.xxx:5000 --protocol=http --enable-threads
```

> **Fix and Tips:** If you meet some problems, please check you installed _MongoDB_ and _uwsgi-plugin-python_. Check errors during the installation process. Try to remove any _*.pyc_ in the current folder. Try to logout and login to see if that's not a problem of paths updating.


> **Note:** It exists many parameters to configure and optimize uWSGI; please see: [uWSGI project](http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html).


### Use pre built scripts

The project repository includes a sample / default application start script: `bin/run.sh`. This script includes an uWSGI default command line that starts the backend and make it listen on all interfaces, port 5000.

From the project home directory:

```bash
./bin/run.sh
```

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

Then install requirements with _pip_ and launch _setup.py_:

```bash
# If you encounter problems with pip install, please retry with '--user' flag.
pip install -r requirements.txt
sudo python setup.py install
```

## Configuring Alignak modules

The installation setup script created some configuration files in your */usr/local/etc/alignak/arbiter_cfg/modules* directory.

You must configure the backend URL and login information in the three following files:

```bash
cd /usr/local/etc/alignak/arbiter_cfg
sudo vi modules/mod-alignakbackendarbit.cfg
sudo vi modules/mod-alignakbackendbrok.cfg
sudo vi modules/mod-alignakbackendsched.cfg
```

> **Hint:** Currently, you simply have to uncomment the `username` and `password` variables to allow connection to the backend. Check **api_url** in this files too !

Then you need to configure Alignak daemons to inform about the existing modules:

> **Note:** Currently, the scheduler backend module is broken and you should not configure it! Fixes is coming soon for the data retention...

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
# keep repos in backend-import
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


> **Fix and Tips:** A detailed documentation is available here: [Doc alignak-backend-import](http://alignak-backend-import.readthedocs.io/en/latest/index.html).

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

You can now proceed to install:

```bash
pip install -r requirements.txt
sudo python setup.py install
# Make run.sh executable
chmod +x run.sh
```

Then go back to our folder for apps and copy settings file:

```bash
cd ~/app; mkdir webui; cd webui
cp ~/repos/webui/etc/settings.cfg settings.cfg
```

The main _settings.cfg_ file is located in */etc/alignak_webui* (or */usr/local/etc/alignak-webui*). The application will read this main file and overload its variables with the one found in the *./settings.cfg* and *./etc/settings.cfg* files if they exist.

You can set several configuration in _settings.cfg_ but you must set your backend URL:

```conf
# settings.cfg
alignak_backend = http://127.0.01:5000
```

## Launching alignak-webui

Now that you have installed alignak-webui, you must start the Web application.

Like the Alignak backend, the WebUI is a Python WSGI compliant application and it exists several solutions to make it run...

### Create your own application starter

Be back to _~/app/webui_ folder and type:

```bash
# create symlink, like this update when updating repos
ln -s ~/repos/webui/bin/run.sh ~/app/webui/run.sh
cp ~/repos/webui/bin/alignak_webui.py alignakwebui.py
```

Then launch the application like this:

```bash
# Replace xxx.xxx.xxx.xxx by IP of your server
uwsgi --plugin python --wsgi-file alignakwebui.py --callable app --socket xxx.xxx.xxx.xxx:5001 --protocol=http --enable-threads
```

> **Fix and Tips:** If you meet some problems, please check you installed _MongoDB_ and _uwsgi-plugin-python_. Check errors during the installation process. Try to remove any _*.pyc_ in the current folder. Try to logout and login to see if that's not a problem of paths updating.


> **Note:** It exists many parameters to configure and optimize uWSGI; please see: [uWSGI project](http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html).


### Use pre built scripts

The project repository includes a sample / default application start script: ``bin/run.sh``. This script includes an uWSGI default command line that starts the WebUI and make it listen on all interfaces, port 5001.

From the project home directory:

```bash
./bin/run.sh
```

## Use Alignak WebUI

Now your WebUI has started and you can reach : http://xxx.xxx.xxx.xxx:5001 on your browser.

You can log in with the default following credentials:

* username: admin
* password: admin

The application builds a log file. The configuration for this log is defined in the _settings.cfg_ file, `logs` section. If the user rights permit, the log file is (as default) stored in the */var/log/alignak-webui* (or */usr/local/var/log/alignak-webui*) directory; else, the log can be tailed in the current folder:

```bash
tail -f ~/app/alignak-webui/alignak-webui.log
```

## VM-Test

If you make install on a Virtual-Machine (with VMWare as example) and you keep localhost (`127.0.0.1:5001`) as default adress, your webui can't be reach at http://127.0.0.1:5001. Cause this is your local loop.

But you can bypass that with **bind_adress**. Open a new terminal and type following command:

```bash
ssh -L 5001:127.0.0.1:5001 login@ip_vm_test
```

Where *ip_vm_test* is your IP test server (and not his local loop !). Then simply open [http://127.0.0.1:5001](http://127.0.0.1:5001) in your favorite browser.

# Additional tools

There is other tools to complete Alignak Suite. Here is the list :

* [Alignak-App](http://alignak-app.readthedocs.io/en/develop/index.html): App Indicator for Alignak who display notifications when something's wrong.

# WIP
