---
layout: post
title: Install GLPI-Monitoring
lang: en
ref: glpimonitoring
modified:
description: How to install glpi-monitoring
tags: [tutoriel, glpi, shinken, monitoring]
image:
  feature:
  credit:
  creditlink:
author:
comments: true
share:
date: 2016-03-10T10:25:10+01:00
---

# Introduction

This tutorial is for people who had a [GLPI](http://glpi-project.org/) server and a [Shinken](http://www.shinken-monitoring.org/) installation; and those who want monitoring our server / computer in GLPI. If you don't have this kind of software installed, please read following tutorials:

* [Install Shinken](/2016/03/shinken-install)
* [Install GLPI](/2016/03/glpi-install)

To have already data in GLPI, you can do it by hand or use an automate tool, like [FusionInventory](/2016/03/install-fusion-inventory). it will allow you to have hosts to monitor.

All below configurations had be done on Shinken 2.4.2 and GLPI 0.90.1 servers. It is therefore advisable to have similar servers. However, most configurations are expected to be virtually identical, except for versions of the plugins used.

> **Note:** You can not import hosts already created in Shinken files to GLPI!

# Prerequisites

To follow this tutorial, you must have:

* A GLPI server, with root access or an account who had required access on GLPI installation.
* A Shinken server with root access.
* DNS records that allow two servers mentionned above to communicate.

It is best also have a minimum of hosts/servers in GLPI to monitor.

# GLPI Plugins

At the moment, we already install plugins required by GLPI to communicate with Shinken. GLPI will need 2 plugins:

* [glpi_Monitoring](https://github.com/ddurieux/glpi_monitoring) : this plugin will display servers you will monitor in Shinken.
* [web_services](https://forge.glpi-project.org/projects/webservices) : this plugin will allow both server to communicate.

For this to be practical, I suggest doing a `glpi_plugin` directory in your` HOME`, able to come back later in future updates. I also recommend using [Git] (https://git-scm.com/) to get your plugins easily.

## Download glpi_monitoring

To get this plugin, simply clone the repository and switch to the wanted `tag` (here `0.90+1.0`):

```bash
git clone https://github.com/ddurieux/glpi_monitoring.git
cd glpi_monitoring
git checkout 0.90+1.0
cd ..
mv glpi_monitoring/ monitoring
sudo cp -R monitoring/ /var/www/glpi/plugins/
sudo chown -R www-data:www-data /var/www/glpi/plugins/monitoring
```

Now, your plugin is ready.

Maybe you'll get the following error in GLPI:

`PHP Fatal error:  Call to undefined function curl_init()`

This means that you are missing the extension `curl` of `php5`. In that case:

```bash
sudo apt-get install php5-curl
```

This solves the problem.

## Download web_services

This plugin is available on GLPI forge:

```bash
wget https://forge.glpi-project.org/attachments/download/2099/glpi-webservices-1.6.0.tar.gz
tar xzvf glpi-webservices-1.6.0.tar.gz
sudo cp -R webservices/ /var/www/glpi/plugins/
sudo chown -R www-data:www-data /var/www/glpi/plugins/webservices/
```

Web_services is ready.

> **Note:** GLPI will recognize the plugin through its folder name and other files. Be careful not to change the folder names of your plugins unless GLPI ask you!

## Enable plugins

Now go to your GLPI interface and log in.

If you go to the menu **Configuration => Plugins** you should see your two newly installed plugins. It is possible that the **web_services** module will display a message: `Installing PHP incorrect. Requires xmlrpc` module! This is normal if your server does not have the extension `xmlrpc` installed.

```bash
sudo apt-get install -y php5-xmlrpc
sudo service apache2 restart
```

Refresh your page, you should not have an error.

Now you can click the buttons **Install** of each plugin and then the **Activate** button. That's it, your plugins are ready to be used. If you go on **Plugins => Monitoring** you will see only a blank page. This is normal for this time it has nothing to show but we will give him something to work.

# Add the necessary users in GLPI

You'll have to create a user in MySQL and GLPI, so they can connect to the _Web Services_ to retrieve its configuration.

* For GLPI: go to **Administration => Users** and add an account `shinken`.
* For MySQL, follow below steps:

Connect to MySQL, on `glpi` Database:

```bash
mysql -u root -p glpi
```

And type following commands (and obviously change password):

```mysql
GRANT select,update ON glpi_plugin_monitoring_services TO shinkenbroker IDENTIFIED BY 'password';
GRANT insert ON glpi_plugin_monitoring_serviceevents TO shinkenbroker;
GRANT select,update ON glpi_plugin_monitoring_servicescatalogs TO shinkenbroker;
```

> **Warning:** if your Shinken and GLPI server are not on same machine, you must define the username with his machine: `'shinkenbroker'@'ip_shinken_server'`!

Type `CTRL-D` or `exit` to go out of MySQL.

Your two users are set.

# Configure Web Services

Go to GLPI interface and **Configuration => Webservices** menu. Add a service with following parameters:

* Name: Shinken
* Active services: Yes
* Enable compression: No (Not tested this feature so far)
* Draw connections: No (turn on if you want to keep track of connections)
* Debug: No (Turn it on when debugging)
* Reason SQL services. * `.*`
* IPv4 address range: set the address of your server Shinken in the 2 boxes.
* IPv6 address: if you have activated the IPv6.
* User ID: leave blank in this case.
* Password: leave blank in this case.

Then click on **Add** button to save your configuration.

# Shinken Modules

Now, let configure Shinken modules. You'll have to set  **Arbiter** and **Broker** daemons to let them connect to GLPI and his Database.

As user `shinken` on the right server (Shinken):

```bash
shinken install import-glpi
shinken install glpidb
shinken install ws-arbiter
```

## Configure import-glpi

You must have an `import-glpi.cfg` configuration file in your `modules` folder, open it and write depending on your configuration:

```conf
define module {
    module_name     import-glpi
    module_type     import-glpi
    # URI of Web service for GLPI
    uri             http://ip_server/glpi/plugins/webservices/xmlrpc.php
    # If you had a valid DNS, point to the right address:
    # uri             http://glpi.domain.com/plugins/webservices/xmlrpc.php
    # Credentials you had set before:
    login_name      shinken
    login_password  password
}
```

Save and quit.

Then you must add it in Arbiter (`vi arbiters/arbiter-master.cfg`):

```conf
[...]
address     0.0.0.0     # Change localhost to 0.0.0.0 display system state in GLPI.
modules     import-glpi
[...]
```

Save and quit. Now, Shinken can connect to GLPI.

## Configure glpidb

You must first change the module configuration:

```conf
define module {
    module_name     glpidb
    module_type     glpidb
    host            ip_serveur     # Name of GLPI server or his IP
    port            3306
    database        glpi           # Database name of GLPI
    user            shinkenbroker  # User created in MySQL
    password        password       # User password
[...]
}
```

Save and quit. Now add this module to Broker (`vi brokers/broker-master.cfg`):

```conf
[...]
modules    webui2,glpidb
[...]
```

Save and quit this file. Shinken can now connect to GLPI database.

## Configure ws-arbiter

For this module, there are no special modification to perform. However if in GLPI you have defined a password and a username in `Webservices` plugin you will have the same data in the configuration file (`modules/ws_arbiter.cfg`).

But you still have to indicate in Shinken Arbiter (`vi arbiters/arbiter-master.cfg`):

```conf
[...]
modules     import-glpi,ws-arbiter
[...]
```

Save and quit again.

# Check that everything works

Now, all your configuration are ready. Just restart Shinken and GLPI:

On Shinken server:

```bash
sudo service shinken restart
```

On GLPI server:

```bash
sudo service apache2 restart
```

And now, you must have the following window if you go in GLPI monitoring page:

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_monitoring.png" alt="">
    <figcaption>GLPI Monitoring</figcaption>
</figure>

You should see a green dot in front of `State system` indicating that all Shinken daemon works as well as `ping`. You can see that you can restart Shinken from the GLPI interface. If you restart Shinken from here a GLPI notification will appear: `Communication with Shinken works...`.

## Check your logs

If you encounter any problems, remember to check your logs!

On Shinken server:

```bash
sudo tail -f /var/log/shinken/*.log | grep ERROR
```

You can also test your `xmlrpc` connection via the following command as **shinken user**:

```bash
shinken-arbiter -v -c /etc/shinken/shinken.cfg
```

For GLPI, check apache2 logs and GLPI:

```bash
sudo tail -f /var/log/apache2/*error.log
# or in files folder of GLPI
sudo tail -f /var/www/glpi/files/_log/*.log
```

If it works, you will set your orders, components and component catalogs (or services). They will then be available in the various menus (state machine, resources, metrics, etc ...).

# Configure a catalog

## Commands

**Note:** the nagios plugins (and commands) were normally installed previously in the [Shinken](/2016/03/shinken-install) tutorial.

The command defined in **glpi_monitoring** are not necessarily the same in your installation. You must check it.

As a reminder, you can st your `$PATH` in: `SHINKEN_DIR/resource.d/paths.cfg` (on your Shinken server).

It is also better to test the command locally before to see if it works well. For example :

```bash
user@shinken:~$./check_snmp_storage.pl -H 127.0.0.1 -C public -m / -w 80% -c 90% -G
/dev: 0%used(0GB/0GB) /: 20%used(11GB/57GB) (<80%) : OK
```

## Components

Once the command defined, you must associate a `component` to her. Fill the following fields:

* Provide a name
* Select the command you want
* Select a graphic template. Most of the templates are already well defined. If you lack a graphic template, create one:
  * Just put your order in `Example of perfdata for this control` field and click **Save**. The plugin will generate your own template.
* Set a Control: it is used to determine how many times shinken will try this command before changing state.
* Active Control => Yes
* Passive Control => Yes
* Control Period: Select period.

Then save!

## Component Catalog

Just create a catalog component to all of this. The catalog will be set according to your wishes: multiple hosts through catalogs or not, several components, etc ... up to you to see how you then want to monitor your hosts. However the process will look something to the following:

* Give it a name
* Add one (or more) component(s)
* Add a Host: you can set a rule to import dynamically or choose a static host.
* Add a contact: You can also add a contact to be warned if your _state_ change (UP, DOWN, WARNING...).

Once your catalog is finish, Shinken should normally restart automatically when you click **Add** button.

# Conclusion

Glpi-Monitoring is not necessarily easy to configure and install; including the complete installation of all these servers.

Take the time to check your settings and your logs to make sure that there is no errors. Do not hesitate to contact the development team on forums or IRC channels.

Also keep in mind that we are mostly volunteers for all these software and we can not always respond quickly.


