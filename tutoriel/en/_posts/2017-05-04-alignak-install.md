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

**WARNING:** Alignak is still under development but is yet stable enough! The first release is however very close.

During this tutorial, we'll install Alignak and its satellites, who are written mainly in Python. Despite Alignak team is trying to make it an easy to set-up monitoring solution, Alignak and its satellites require a minimum knowledge with Linux and being resourceful.

This tutorial comes in addition to the demo tutorial provided by the Alignak team: [Alignak Demo](https://github.com/Alignak-monitoring-contrib/alignak-demo).

Almost all of Alignak's repositories have stable versions available via `pip`. But I'll try to provide each repository URL each time you install a new software.

# Prerequisites

> **Note:** This tutorial was run on an Ubuntu 16.04 LTS server. Other Linux distributions are also compatible.

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

## Optional requirements

This tutorial currently uses **screen** because the default scripts provided by Alignak use it:

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

# Alignak base

> **Note:** You have to know that all Alignak components need a root account (or sudo privileges) to get installed.

Alignak need also an **alignak** user. Don't create it yet, we'll create after with a script.

Many components will be installed with pip, only the Alignak framework is installed with Git. If you want to install from the repository also, it is recommended to use only the **develop branch**. Alignak team makes sure that the develop branch of all the repositories is always a stable version.

## Alignak framework

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

Pay attention to the end message, it will already give you a lot of indications on what is good or not.

As you can see, Alignak installer did not found any **alignak** user / group and reports also that you've to set some permissions on specific folders.

To correct this, run the following script:

```bash
sudo ./dev/set_permissions.sh
```

You'll have the following output:

```
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

> **Note:** Alignak framework is enough to check the monitored system and get information about the detected problems thanks to the notifications. But we need more than this... what about a friendly user interface, logs retention, performance data...? 

## Alignak Backend

> **Repos:** [Alignak Backend](https://github.com/Alignak-monitoring-contrib/alignak-backend)

The Alignak backend is the core of the monitoring solution. To run it, you'll need to have a running Mongo database (see above).

To install alignak-backend:

```bash
sudo pip install alignak-backend
```

You can also install the **alignak-backend-import** tool. Alignak-backend-import will be used to import your existing Nagios/Shinken configuration files later.

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

The WebUI is the the Web User Interface for the Alignak solution. To install it, run the following command:

```bash
sudo pip install alignak-webui
```

In fact, if you want to use Alignak in a comfortable way you're yet ready. But Alignak still has many components that are useful.

# Checks

> **Note:** You do not have to **install all the checks** below. Choose only what you are interested in and match your infrastructure.

The most known is [Nagios Plugins](https://www.nagios.org/projects/nagios-plugins/). You can find a tutorial on this website on how install and use them and also for SNMP plugins:

* [Install Nagios and SNMP plugins](/2016/12/install-nagios-and-snmp-plugins)

There are many other checks packages available on the [Alignak monitoring contributions Github organization](https://github.com/Alignak-monitoring-contrib):

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

You can add more later if necessary.

# Alignak Modules

Alignak modules add some features to the Alignak framework. Some are necessary for the essential alignak features (like monitoring logs or performance data), others are fully optionnal.

## Backend Module

> **Repos:** [Alignak Module Backend](https://github.com/Alignak-monitoring-contrib/alignak-module-backend)

The Alignak Backend modules are needed for getting the configuration from a database, storing the monitored items live state, storing the logs, sending performance data to timeseries databases...

```bash
sudo pip install alignak-module-backend
```

## Logs Module

> **Repos:** [Alignak Logs Backend](https://github.com/Alignak-monitoring-contrib/alignak-module-logs)

The logs module creates log files for all the monitoring events: alerts, notifications... This module is a must-have to track all your monitored system events.

```bash
sudo pip install alignak-module-logs
```

## Web services

> **Repos:** [Alignak Web Services](https://github.com/Alignak-monitoring-contrib/alignak-module-ws)

The Web Services module exposes Web services to get information from Alignak (logs, daemons status), notify external commands to Alignak or push checks results to Alignak. This module is also used by the Alignak Web UI to interact with the monitored systems thanks to commands.

```bash
sudo pip install alignak-module-ws
```

## Notifications

> **Repos:** [Alignak Notifications](https://github.com/Alignak-monitoring-contrib/alignak-notifications)

This package will add commands to send notifications using HTML email, XMPP, ... very useful if you want to be notified when something goes wrong:

```bash
sudo pip install alignak-notifications
```

## Other Modules

> **Repos:** The following modules are all available under [Alignak monitoring contributions Github organization](https://github.com/Alignak-monitoring-contrib).

These modules add other commands or services.

```bash
# Collect passive NSCA checks
sudo pip install alignak-module-nsca
# Write external commands (Nagios-like) to a local named file
sudo pip install alignak-module-external-commands
# Improve NRPE checks
sudo pip install alignak-module-nrpe-booster
```

# Configure Alignak

Now you normally have everything you need to get started.

As you will see, most of the configuration will be located in `/usr/local/etc`. Inside this folder, you'll find the _alignak_, _alignak-backend_ and _alignak-webui_ folders.

Alignak packages and modules will install there configuration files inside `/usr/local/etc/alignak` (like alignak-notification for example). With this system alignak developers want the configuration of Alignak and its satellites to be centralized maximum.

> **Note:** All alignak configuration files are widely commented and detailed for each options ! Read carefully before making any changes.

## Check-list for your installation

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

As you can see some files in alignak folder are missing _alignak_ user / group and the same for the **log** folder.

To solve that, run the following commands:

```bash
cd ~/repos/alignak
sudo ./dev/set_permissions.sh
```

Now your installation is `OK` and ready to get adapated to your needs!

We'll begin with the Backend and WebUI folder which are the more simple to set-up.

## Configure Alignak Backend

You will find 2 files in the *alignak-backend* folder:

* uwsgi.ini
* settings.json

Right now, edit the following file:

```bash
sudo vi /usr/local/etc/alignak-backend/uwsgi.ini
```

In this file you can see that backend is listening by default on `0.0.0.0:5000`. You can change IP and Port but don't forget after to report this change to the other files (alignak backend modules configuration files)!

The JSON file contains some other configuration parameters essentially for the Backend data, Mongo or Timeseries databases.

## Configure Alignak WebUI

In *alignak-webui* folder, you'll find 3 files:

* uwsgi.ini
* settings.cfg
* logging.json

Open the `uwsgi.ini` file. You will see that this file looks much like the backend one. WebUI is listening on `0.0.0.0:5001` by default. Personnaly I prefer to use port 80, so:

```conf
http-socket = 0.0.0.0:80
```

Then open the main WebUI configuration file: `settings.cfg`. If you've modified IP and port of your WebUI, report your change here:

```ini
[bottle]
host = 0.0.0.0
port = 80

; If you've changed backend IP:PORT, search, update and uncomment alignak_backend line
; alignak_backend = http://127.0.0.1:5000

; If you installed Alignak Web Services module, search, update and uncomment alignak_ws line
; alignak_ws = http://127.0.0.1:8888
```

There are many options available to customize your WebUI. Read the comments that explain what they are used for and how to configure the options.

The last file if for log formatting. You can have a look if you want, but it is not necessary to modify it.

## Folders and files

Now we have to define Alignak itself. If you go in the folder `/usr/local/etc/alignak/`, you'll see this:

```bash
cd /usr/local/etc/alignak/
ls -la
-rw-rw-r-- 1 alignak alignak 6445 Jan 29 21:20 alignak.backend-run.cfg
-rw-rw-r-- 1 alignak alignak 7756 May  4 06:44 alignak.cfg
-rw-rw-r-- 1 alignak alignak 3808 May  4 06:44 alignak.ini
drwxrwxr-x 8 alignak alignak 4096 May  4 06:44 arbiter/
drwxrwxr-x 2 alignak alignak 4096 May  4 06:44 certs/
drwxrwxr-x 2 alignak alignak 4096 May  4 06:44 daemons/
drwxrwxr-x 3 alignak alignak 4096 May  4 06:44 sample/
```

The `alignak.cfg` file is the configuration file you'll import in the backend of Alignak. It mainly defines the files that will be imported or not in the backend. Currently you don't need to modify something in this file, unless you do not use the default directories.

The `alignak.backend-run.cfg` is a sample file like the one above. This file is intended to be used when you want to run Alignak with the configuration you stored in the backend. 

The `alignak.ini` file is only used by the scripts provided in the *dev* forlder of the repository. It is not used by Alignak so do not care about them;)

Folder **arbiter**: this folder contains all main configuration files. Here you will define your hosts, your services, your check commands, as well as the configuration of the modules.

Folder **certs**: it is used to store your certificates if you are using SSL connection between the Alignak daemons.

Folder **daemons**: contains the daemon _ini_ files, which allow to define the folders, the pid, the port,...

Folder **sample**: this last folder contains sample configuration files for your services, hosts, commands,...

## Configure daemons

First we will configure the daemons. Into the `arbiter/daemons` folder, you can find one `.cfg` file for each Alignak daemon.

You must at least add the modules of the Backend and the other modules that you've installed. Here is sample examples for each daemons:

* Arbiter

```conf
# Address of the daemon
address     127.0.0.1

# Add the modules you have installed.
modules     backend_arbiter
```

* Broker

```conf
# Address of the daemon
address     127.0.0.1

# Add the modules you have installed.
modules     backend_broker, logs
```

* Poller

```conf
# Address of the daemon
address     127.0.0.1

# If you install booster modules, add them here !
```

* Reactionner

```conf
# Address of the daemon
address     127.0.0.1

# There is currently no modules available for this daemon.
```

* Receiver

```conf
# Address of the daemon
address     127.0.0.1

# If you install web-services, nsca or external-commands, add them here !
```

* Scheduler

```conf
# Address of the daemon
address     127.0.0.1

# Add the modules you have installed.
modules     backend_scheduler
```

For each file, you can also set _timeout_, *data_timeout*, *max_check_attempts* and *check_interval* and other settings, like _spare_ or _ssl_. For the moment, do not care about this...

## Configure modules

Now that you have linked your modules to the daemons, you must configure them in `arbiter/modules/` folder. 

The configuration in this folder is almost the same. Be sure to update the `api_url` if your backend is not on **127.0.0.1** by default. Here is an example

```conf
# Change address according to your backend configuration
api_url                 http://127.0.0.1:5000

# Use your token instead of username and password
# token                 xxxxxxxxxxxxx-xxxxxxx-xxxx-xxxxxxxxxxxx
# For the first import, you can let password by default
# It is recommended to change it afterwards
username                admin
password                admin
```

## Other configurations

We have already seen what is inside _daemons_ and _modules_ folder. But if you've installed other alignak components, like _notifications_, you will find them in the `packs` folder.

For the _alignak-notifications_ case, you'll have the following:

```bash
ls -la arbiter/packs/
drwxrwxr-x 2 alignak alignak 4096 May  4 08:08 notifications/
-rw-rw-r-- 1 alignak alignak  128 May  4 06:44 readme.cfg
drwxrwxr-x 2 alignak alignak 4096 May  4 08:08 resource.d/
```

Inside notifications, you'll have `notification-ways.cfg` file which defines the way users will be notified. You must add your choosen notification-ways in your contacts if you want to receive notifications.

You can see in this file that you have commands. They are defined in the `commands.cfg` file next to it.

Finally, inside the `resource.d` folder, you have `notifications.cfg` file where you can add your SMTP settings.

> **Note:** this pack requires an SMTP server for the mail notifications to be sent out. If none is available you will get WARNING logs and the notifications will not be sent out, but alignak will run anyway.

## Objects

The `arbiter/objects` folder already contains default configurations commands, contacts, hosts,...

Normally, you have the localhost host already define. Let's see what it looks like by opening the following file: `objects/hosts/localhost.cfg`.

```cfg
define host{
use                     generic-host
contact_groups          admins
host_name               localhost
address                 localhost
}
```

* use: tell to alignak which template this host will use, located in `arbiter/templates` folder.
* contact_groups: tell with which contact groups this host is related. If a user is not in this group, he will not be related to this host for the notifications.
* host_name: the name of host
* address: the adress of host (IP or FQDN). Be sure your server can join your DNS to resolve FQDN !

Currently, localhost has no service defined, we will add one.

If you've install some **checks**, they are normally located in one of the following folders: */usr/lib/nagios/plugins/* (for Nagios-plugins), */usr/local/libexec/monitoring-plugins/* (for Monitoring plugins) and */usr/local/var/libexec/alignak/* (for alignak-checks).

For the following example, we will use the [alignak-checks-snmp](https://github.com/Alignak-monitoring-contrib/alignak-checks-snmp) and the command `check_snmp_storage.pl`. If you have not installed this plugins, you can do it now:

```
sudo pip install alignak-checks-snmp
sudo apt-get install libsnmp-perl libnet-snmp-perl
```

> **Note:** When you install these checks, it also installs pre-configured files in `arbiter/packs/snmp/` folder. They could therefore be used rather than created, as in the following example.

Personally, I define services in hosts configuration files (so in `objects/hosts` folder) and commands in the `objects/commands` folder. But you can do as you want.

> **Note:** the nagios-like configuration files can be organized the way you want. The most important is that the added files are in a `cfg_dir` folder defined in the file `alignak.cfg` !

We first add a command:

```bash
sudo vi arbiter/objects/commands/check-storage.cfg
```

```cfg
define command {
	command_name	linux_check_snmp_storage
	command_line	$PLUGINSDIR$/check_snmp_storage.pl -H $HOSTADDRESS$ -C $_HOSTSNMPCOMMUNITY$ -f -m / -w 85 -c 90 -S0,1 
}
```

Then we had this command to a service:

```bash
sudo vi arbiter/objects/hosts/localhost.cfg
```

```cfg
define host {
	use             generic-host
	contact_groups  admins
	host_name       localhost
	address         localhost
}

define service{
	service_description   Disks
	use				generic-service
	host_name		localhost
	check_command	linux_check_snmp_storage
}
```

That's all ! If we try to sum-up, we have an host which uses a template, and which has a service which uses a template and this last has a command. **A service must always have an associated host and a check command!**

You have noticed that there are variables (macros) in these configuration files. They are usually defined in other configuration files, such as templates. For the `$SNMPCOMMUNITYREAD$`, you can find her in `arbiter/packs/resource.d/snmp.cfg` file.

Each variable **must** follow the following syntax: `$MY_VARIABLE$` !

# Start Alignak

Now we are ready to start our alignak server. We'll use **screen** tool we have installed at the beginning of the tutorial. To make things easier we'll create scripts for that.

Return to your home folder or in a folder you're sure and make an `alignak_cmds` folder. Don't create it in the alignak repos folder or install folders of alignak ! Otherwise, it will be erased during updates !

```bash
cd ~;
mkdir alignak_cmds; cd alignak_cmds/
```

## Start/Stop backend

First we have to start the backend. Be sure your **mongod is running** before !

To create the script, run the following commands:

```bash
# Start file
cat > alignak_backend_start <<EOF
#!/bin/sh
echo "Starting Alignak backend..."
screen -d -S alignak-backend -m sh -c "alignak-backend-uwsgi"
sleep 1
netstat -tulpen | grep 5000
echo "Started"
EOF

# Stop file
cat > alignak_backend_stop <<EOF
#!/bin/sh
echo "Stopping Alignak backend..."
kill -INT \`cat /tmp/alignak-backend.pid\`
sleep 1
echo "Stopped"
EOF

# Make executable
chmod +x alignak_backend_*
```

Now simply start backend:

```bash
sudo ./alignak_backend_start
# Output
Starting Alignak backend...
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      0          30247       2697/uwsgi      
Started
```

You can check your logs in _/usr/local/var/log/alignak-backend/backend-error.log_. If there is any error, you'll see it inside.

# Feed Backend

Now that the backend is running, you can feed it with your configuration files. We'll use `alignak-backend-import` tool.

Launch this command by adapting it to your configuration:

```bash
sudo alignak-backend-import -d -b http://127.0.0.1:5000 -u admin -p admin /usr/local/etc/alignak/alignak.cfg
```

You can see the comman line syntax with: `alignak-backend-import -h`.

Here is the output:

```bash
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
alignak-backend-import, version: 0.9.0
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Backend URL: http://127.0.0.1:5000
Dry-run mode (check only): False
Delete existing backend data: True
Updating backend data: False
Allowing duplicate objects: False
Default host location: {'type': 'Point', 'coordinates': [46.60611, 1.87528]}
Importing configuration: ['/usr/local/etc/alignak/alignak.cfg']
Loading daemon configuration file (None)...
No daemon configuration file specified, using defaults parameters
[...]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
alignak-backend-import, inserted elements: 
 - 25 command(s)
 - 2 host(s)
 - 9 host_template(s)
 - no hostdependency(s)
 - no hostescalation(s)
 - 3 hostgroup(s)
 - 1 realm(s)
 - 1 service(s)
 - 35 service_template(s)
 - no servicedependency(s)
 - no serviceescalation(s)
 - 2 servicegroup(s)
 - 4 timeperiod(s)
 - 3 user(s)
 - 1 user_template(s)
 - 2 usergroup(s)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Global configuration import duration: 3.35982680321
```

The alignak-backend-import tool will tell you what is imported in backend and will notify you if there is any error.

## Start/Stop alignak daemons

Now that you've the backend is running and filled, you can start alignak daemons. You already have scripts inside the _alignak_ repository you've cloned.

```bash
cd ~
sudo ./repos/alignak/dev/launch_all.sh
---
Alignak ini configuration file found in /usr/local/etc/alignak folder
Alignak ini configuration file: /usr/local/etc/alignak/alignak.ini
---
---
Alignak daemon: SCHEDULER_MASTER
---
Alignak configuration file: /usr/local/etc/alignak/alignak.cfg
Alignak extra configuration file: 
---
Daemon script: SCHEDULER_MASTER_DAEMON = alignak-scheduler
Daemon configuration: SCHEDULER_MASTER_CFG = /usr/local/etc/alignak/daemons/schedulerd.ini
Daemon debug file: SCHEDULER_MASTER_DEBUGFILE = /usr/local/var/log/alignak/scheduler-debug.log
---
Launching the daemon: SCHEDULER_MASTER with configuration file: /usr/local/etc/alignak/daemons/schedulerd.ini
Launching the daemon: SCHEDULER_MASTER
Daemon scheduler is in daemon mode
Loading daemon configuration file (/usr/local/etc/alignak/daemons/schedulerd.ini)...
[...]
INFO:alignak.daemon:Changing working directory to: /usr/local/var/run/alignak
[2017-05-05 05:51:03 PDT] INFO: [alignak.daemon] Using working directory: /usr/local/var/run/alignak
INFO:alignak.daemon:Using working directory: /usr/local/var/run/alignak
[2017-05-05 05:51:03 PDT] INFO: [alignak.http.daemon] Opening HTTP socket at http://127.0.0.1:7770
INFO:alignak.http.daemon:Opening HTTP socket at http://127.0.0.1:7770
[2017-05-05 05:51:03 PDT] INFO: [alignak.daemon] Daemonizing...
INFO:alignak.daemon:Daemonizing...
```

This script will launch all the daemons, like says in the [Alignak Docs](http://docs.alignak.net/en/develop/03_how_it_work/run_daemons.html#running-alignak), by following the following syntax:

```bash
# For broker, poller, reactionner, receiver and scheduler
alignak-<daemon> -c /usr/local/etc/alignak/daemons/<daemon>d.ini
# For arbiter (the last daemon that should start!)
alignak-arbiter -c /usr/local/etc/alignak/daemons/arbiterd.ini -a /usr/local/etc/alignak/alignak.cfg
```

If there are some errors, you will find logs in `usr/local/var/log/alignak/` for each daemons. If you installed and configured the `module-logs`, you'll have `monitoring-logs.log` in addition.

# Start/Stop alignak webui

Having a backend and running daemons is cool, but viewing them in an interface should be better. For that, you have the Alignak WebUI.

To start Webui, we'll do the same as for the backend, we make script with _screen_ tool. Update port in "start" command according to your configuration.

```bash
# Start file
cat > alignak_webui_start <<EOF
#!/bin/sh
echo "Starting Alignak WebUI..."
screen -d -S alignak-webui -m sh -c "alignak-webui-uwsgi"
sleep 1
netstat -tulpen | grep :80
echo "Started"
EOF

# Stop file
cat > alignak_webui_stop <<EOF
#!/bin/sh
echo "Stopping Alignak WebUI..."
kill -INT \`cat /tmp/alignak-webui.pid\`
sleep 1
echo "Stopped"
EOF

# Make excutable
chmod +x alignak_webui_*
```

Then simply start WebUI:

```bash
sudo ./alignak_cmds/alignak_webui_start
```

Then, if you've define your WebUI with your server IP, you should be able to see login page on : http://ip_server.

Use the login of a user defined in your configuration, else you can use the default *admin*/*admin* of the default user created on the first backend start.


