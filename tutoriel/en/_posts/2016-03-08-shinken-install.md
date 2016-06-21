---
layout: post
title: Install Shinken
lang: en
ref: shinken
modified:
description: How to install Shinken
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

In this tutorial, we'll sse how to install [Shinken](http://www.shinken-monitoring.org/index.php) on a Linux server. Either way, installation of Shinken is easy and can be done in several ways. The [Documentation](http://shinken.readthedocs.org/en/latest/index.html) is very well made and enjoyable to read. But, here we show installation from source and the way to configure Shinken and his web interface.

# The various installation of Shinken

If you have seen documentation, there's 3 different ways to install Shinken:

* via [Pip](https://pip.pypa.io/en/stable/) (module installer of Python) who's often up to date.
* via package who's not often up to date.
* via sources of [Shinken repository](https://github.com/naparuba/shinken) we'll use in this tutorial.

You are free to install Shinken as you like. But the **most important** is to use only **one way** for installation and updates of Shinken ! In any case, you have to mix several types of install under pain of seeing your server crash, have bugs or not work at all! I invite you also to write a small file in your installation directory to note how you installed Shinken and its various modules / libraries.

# Prerequisites

Shinken is written in Python, he'll need a Python version to work (Version 2.6 or more and if possible 2.7 for better performances. He'll needs too [python-pycurl](http://pycurl.io/) and [setuptools](https://pypi.python.org/pypi/setuptools/).

Here is several steps for this dependencies, knowing this tutorial was made on Ubuntu Server 14.04.4 LTS. Adapt it depending on your configuration.

## Install Python

To install Python, nothing's easier:

```bash
sudo apt-get update
sudo apt-get install python
```

In general, on Ubuntu Server, Python 2.7 is already install by default (and several modules too). That's possible, it'll be not available for your distribution via package. Ask then to the site of your distribution or search on to install via other way.

### Install PyCurl

To install PyCurl, just install from package :

```bash
sudo apt-get install python-pycurl
```

### Install setuptools

The same thing here :

```bash
sudo apt-get install python-setuptools
```

Now, you have all required dependencies installed for Shinken.

# Get sources of Shinken

To retrieve the sources of Shinken and especiallu the last _release_, visit [Shinken Repository](https://github.com/naparuba/shinken) and download it :

```bash
sudo apt-get install -y git
cd ~
git clone https://github.com/naparuba/shinken.git
cd shinken
# You can see different release by typing "git tag -l"
git checkout 2.4.2
```

Now, your Shinken folder is ready, you can launch setup.

> You can of course download with `wget` or via a graphical interface, but I prefer this method because it will update your version of easier Shinken. With Git you just have to "pull" the new versions and get you back on a `tag` later.

# Install Shinken

Before launch install, you must create a user `shinken` :

```bash
adduser shinken
```

Once done, you can type following command :

```bash
cd ~/shinken
sudo python setup.py install
```

Normally the installation should proceed without problems. You will have some `warning` early to signal that it is not Windows. But especially at the end of the installation you will have the following lines:

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

As you can see, Shinken give rights to `shinken` user you've set before, to folder he needs. You have nothing to do for that.

He also points out that the `daemons` of Shinken will perform better if you install the `python-cherrypy3` library. This dependency is optional, but you can install it:

```bash
sudo apt-get install -y python-cherrypy3
```

Here , Shinken is ready for operation. You can enable and start it:

```bash
sudo update-rc.d shinken defaults
sudo service shinken start
```

You should have the following output:

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

Congratulations, your Shinken is now operational and all his daemons are launched. You will be able to monitor the servers in your infrastructure. But to make it more convenient it would be better to install a Web interface.

# Install the Web interface

For seeing a more pleasant way if your guests are well monitored, a web interface is available for Shinken . There are 2 versions of this interface :

* Webui no more any maintained by developers, this interface is to disappear. But it is still functional.
* Webui2 more updated, ergonomic and growing. It is not yet complete but already has everything it takes to be functional.

Log in as **shinken**:

```bash
sudo su - shinken
```

And initialize the CLI Shinken to generate the file `.ini` containing the paths to the different configuration directories:

```bash
shinken --init
```

> **Important:** all commands `shinken` have to be launch as user `shinken` !

You should have now:

```bash
Creating ini section paths
Creating ini section shinken.io
Saving the new configuration file /home/shinken/.shinken.ini
```

> Please note, although both `webui` can coexist I advise you to choose only one of it. If you install both interfaces, set a different port for one of them (other than `7767`!).

## Case 1 : Webui

For Webui interface, you'll install other modules. Type following commands to install them:

* `shinken install webui` -> webui interface.
* `shinken install auth-cfg-password` : authentication module. (Others authentication module exists)
* `shinken install sqlitedb` : storage module for user's data.

When typing each command, you'll have follwoing lines :

```bash
Grabbing : module_name
OK module_name
```

Once modules installed, you'll have to tell to Shinken you want use them. Open file `/etc/shinken/brokers/broker-master.cfg` and found line uncomment where there's `modules` to add **webui**:

```conf
[...]
modules    webui
[...]
```

Then, edit the same way `/etc/shinken/modules/webui.cfg` to add **auth-cfg-password** and **SQLitedb**:

```conf
[...]
modules     auth-cfg-pasword,SQLitedb
[...]
```

Congratulations ! Your **Webui** interface is configured. If all Shinken daemons have been restarted, you should see the following page by going to the address `http://server_ip:7767`:

<figure>
    <img src="{{ site.url }}/images/shinken/shinken_webui.png" alt="">
    <figcaption>Shinken - Écran de connexion Webui</figcaption>
</figure>

## Case 2 : Webui2

For the second interface, **Webui2**, isntallation is different cause she manage herself a lot of things. You've to install additional Python libraries.

But before, install `Webui2`:

```bash
shinken install webui2
```

After, as `root` user, install `pip` and the database management system `mongodb`:

```
sudo apt-get install python-pip mongodb
```

Next libraries can be installed by package, but you certainly do not have the required versions for `webui2`! You must install by **pip**:

```
sudo pip install pymongo>=3.0.3 requests arrow bottle==0.12.8
```

Once all dependencies installed, you have to tell to Shinken to use `webui2` in _broker_ (`vi /etc/shinken/brokers/broker-master.cfg`):

```bash
[...]
modules     webui2
[...]
```

Restart shinken :

```bash
sudo service shinken restart
```

That's done ! Your interface **Webui2** is configured. If you go to `http://ip_serveur:7767`, you'll see the following page:

<figure>
    <img src="{{ site.url }}/images/shinken/shinken_webui2.png" alt="">
    <figcaption>Shinken - Écran de connexion Webui2</figcaption>
</figure>

# Configure user admin

Now you can change password for user `admin` in file `/etc/shinken/contacts/admin.cfg`:

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

Save and quit.

You cancheck your configuration by typing following command:

```bash
sudo service shinken check
```

Restart Shinken to take new configuration. And connect to you web interface with your new credentials.

# Commands and Nagios Plugins

As you can see, on your web interface or in the `scheduler` logs, the only host is **localhost**. Maybe you can see also the error:

```bash
[Errno 2] No such file or directory
```

And host can be `DOWN` for Shinken. Don't worry, your server's working great. It's possible that Shinken can't found command or have wrong path by default.

## Commands and paths

In fact, Shinken needs commands, running (in general) with [snmp](https://fr.wikipedia.org/wiki/Simple_Network_Management_Protocol) protocol to monitor your server.

Before install something, open following file (as user `shinken`):

```bash
vi /etc/shinken/commands/check_host_alive.cfg
```

You should have something similar to:

```conf
define command {
    command_name    check_host_alive
    command_line    $NAGIOSPLUGINSDIR$/check_ping -H $HOSTADDRESS$ -w 1000,100% -c 3000,100% -p 1
}
```

This indicates that the `check_host_alive` order is defined as follows:

* his name: `check_host_alive` (will be display in logs or webui)
* his command: define here by a variable `$NAGIOSPLUGINSDIR$` and a command `check_ping` with parameters.

You have all information to know what's command do. But it still missing definition of variable `$NAGIOSPLUGINSDIR$` !

In fact, she's define in anoteher file `/etc/shinken/resource.d/paths.cfg` (open it):

```config
# Nagios legacy macros
$USER1$=$NAGIOSPLUGINSDIR$
$NAGIOSPLUGINSDIR$=/usr/lib/nagios/plugins

#-- Location of the plugins for Shinken
$PLUGINSDIR$=/var/lib/shinken/libexec
```

As you can see, by default, Shinken expects to have orders in the `/usr/lib/nagios/plugins`! But since for now this folder does not exist and there are no control / file `check_ping`, you report this by setting an error (`[Errno 2] No such file or directory`).

You have to install Nagios plugins.

> Recent version of Shinken may have been solve this problem and put a default command somewhere.

## Install Nagios Plugins

To install Nagios plugins, it's simple, you've just to download it on official website : [Nagios-plugins](https://nagios-plugins.org/).

```bash
cd ~
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
tar -xzvf nagios-plugins-2.1.1.tar.gz
# Now install it
cd nagios-plugins-2.1.1/
./configure --with-nagios-user=shinken --with-nagios-group=shinken
make
sudo make install
```

Your plugins are installed. They may not be installed in the directory `/usr/lib/nagios/plugins`. In this case , search for where they are (`whereis nagios` for example) and edit the file `/etc/shinken/resource.d/paths.cfg` accordingly.

For example :

```conf
# Nagios legacy macros
$USER1$=$NAGIOSPLUGINSDIR$
#$NAGIOSPLUGINSDIR$=/usr/lib/nagios/plugins
$NAGIOSPLUGINSDIR$=/usr/local/nagios/libexec

#-- Location of the plugins for Shinken
$PLUGINSDIR$=/var/lib/shinken/libexec
```

You have just to restart Shinken to apply change :

```bash
sudo service shinken restart
```

Check on the web interface once every daemons restarted. And then normally your `localhost` is `up` (there may be a small delay while Shinken launch well all his daemons)!

Congratulations ! You finally got to monitor a server with Shinken !

# Conclusion

Shinken is fairly easy to implement and just requires a little organization during installation of its modules. You can also find a list of these modules on the [official website](http://shinken.io/browse/modules/updated) with their different configurations. 

Shinken is also capable of displaying graphics on its Web interface or [GLPI](http://glpi-project.org/) through plugin [glpi_monitoring](https://github.com/ddurieux/glpi_monitoring). You can find tutorials here to make this work on GLPI.


