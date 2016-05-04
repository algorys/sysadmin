---
layout: post
title: FusionInventory for GLPI
lang: en
ref: fusioninventory
modified:
description: How to install FusionInventory for GLPI
tags: [tutoriel, fusioninventory, glpi]
image:
  feature:
  credit:
  creditlink:
comments: true
share:
date: 2016-03-14T13:20:05+01:00
---

# Introduction

[FusionInventory for GLPI](https://github.com/fusioninventory/fusioninventory-for-glpi) is a GLPI plugin used to collect data with agents [FusionInventory](http://fusioninventory.org/) installed on devices. 
It update data in GLPI inventory and it's automatic.
Agents can :

* inventory the hardware and software of computers, servers, virtual machines
* inventory the hardware and software of android smartphones
* inventory network devices through SNMP (switch, router, network printer...)
* network discovery (discover all devices connected to the network)
* applications deployment on computers
* wake on lan (wake a computer remotely)
* ESX servers inventoy (on each ESX server or through vcenter)

The agents can be installed with GPO or with scripts and run on many operating systems (Windows, Mac OS X, Linux, *BSD...).

This process can save time and money to make inventory and so the "Sysadmin" life.

# prerequisites

To use this tutorial, you need a server [GLPI](/tuto/glpi-installation/) and it's better to have the version `0.90.1` (the last stable release). 
Anyway, other versions of GLPI can work.

* A `root` / `administrator`  access on this server needed.
* You need too one (or many) Linux server/client to install the agent.
* One (or many) Windows server/client to install an agent.

> It's not necessary to have a Windows and a Linux for this tutorial, but both can be needed.

# How to get the archive

## Simple method

Get the last stable version on the [releases page](https://github.com/fusioninventory/fusioninventory-for-glpi/releases)

```bash
wget https://github.com/fusioninventory/fusioninventory-for-glpi/releases/download/glpi090%2B1.2/fusioninventory-for-glpi_0.90.1.2.tar.gz
tar zxvf fusioninventory-for-glpi_0.90.1.2.tar.gz
sudo cp -R fusioninventory/ /var/www/glpi/plugins/
```

> Be careful, not give rights to apache on this folder (security ^^)

Your folder is ready!


## Developer method with git

To get the plugin FusionInventory for GLPI, you can use git. Install it if not yet:
```bash
sudo apt-get install -y git
```

After, get the git repository and go to the required `tag`(I mean the last release) :

```bash
sudo cd /var/www/glpi/plugins/ && git clone https://github.com/fusioninventory/fusioninventory-for-glpi.git fusioninventory
sudo cd /var/www/glpi/plugins/fusioninventory && git checkout glpi090+1.2
```

> Be careful, not give rights to apache on this folder (security ^^)

Your folder is ready!

# Activate the plugin

Go on web interface of GLPI, log in and go in menu **Configuration => Plugins**. 
Click on **Install** then on **Activate**. Your plugin is now ok.

Now, we need configure it. 
It's not easy to find, go in mernu **Administration => Entities => Root entity** and click on tab `Fusioninventory`. 

> `Root entity` is the main entity by default in GLPI. If your configuration is different, apply changes with it.

In this form, fill the field `URL service access` with your url (or with you IP. Check the url is ok and work!). 
The url to fill is GLPI url, so url must stop to GLPI (example : http://ipserver/glpi).
Then save!

Congratulations, your plugin is ready.

# Install an agent

## Windows

The windows installation in simple, download the client on the[dedicated page](http://forge.fusioninventory.org/projects/fusioninventory-agent-windows-installer/files) and choosing the right `.exe`.

Then, follow the installation process and ut url of your GLPI server:

`http://ipserver/glpi/plugins/fusioninventory/`

or if you have a DNS:

`http://fqdn/plugins/fusioninventory/`

## Linux

For Linux, it's different, need add repositories and install with them or install from sources.

### Installation with repositories

For that, be sure you have a stable version and can update it easily. Get the GnuPG key with this command:

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 049ED9B94765572E
```

> **Note:** The official documentation use the address `keyserver.ubuntu.com` without port and URI. 
However under Ubuntu Server, I need to force port 80 and add `hkp://`. With this the command works.

If you not have tool `lsb-release`, install it:

```bash
sudo apt-get install lsb-release
```

Once this done, we can add the repository in file `sources.list`:

```bash
echo "deb http://debian.fusioninventory.org/debian/ `lsb_release -cs` main" >> /etc/apt/sources.list
```

Update you packages and install the agent

```
sudo apt-get update
sudo apt-get install fusioninventory-agent
```

### Installation with sources

To install with sources, we need install dependencies:

```
sudo apt-get install dmidecode nmap make
```

After, need install "some" `perl` dependencies:

```bash
sudo apt-get install libmodule-install-perl libmodule-build-perl libhttp-server-simple-psgi-perl libhttp-proxy-perl libio-captureoutput-perl libipc-run-perl libnet-snmp-perl libnet-telnet-cisco-perl libpoe-component-client-dns-perl libpoe-component-resolver-perl libtest-compile-perl libtest-deep-perl libtest-exception-perl libtest-most-perl libhttp-server-simple-authen-perl libio-capture-perl libio-captureoutput-perl libpoe-component-client-ping-perl libtest-http-server-simple-perl libtest-mockmodule-perl libtest-mockobject-perl libtest-nowarnings-perl libtest-failwarnings-perl libtest-warnings-perl libfile-copy-recursive-perl libxml-treepp-perl libproc-daemon-perl libproc-pid-file-perl
```

Get the archive, so the 2.3.17 :

```bash
wget https://github.com/fusioninventory/fusioninventory-agent/archive/2.3.17.tar.gz
tar xzvf 2.3.17.tar.gz
cd fusioninventory-agent-2.3.17
```

Then install that:

```bash
perl Makefile.PL
make
sudo make install
```

And voila, you should have after the perl command, a summary of folders used:

```bash
Installation summary
--------------------
prefix: /usr/local
configuration installation directory: /usr/local/etc/fusioninventory
constant data installation directory: /usr/local/share/fusioninventory
variable data installation directory: /usr/local/var/fusioninventory
```

And after `make install`, you will have an import information: where find the agent configuration file!

```bash
if [ -f //usr/local/etc/fusioninventory/agent.cfg ]; then \
        install -m 644 etc/agent.cfg /usr/local/etc/fusioninventory/agent.cfg.new; \
    else \
        install -m 644 etc/agent.cfg /usr/local/etc/fusioninventory/agent.cfg; \
    fi
```

### Perl dependencies

It's possible some dependencies are missing when run agent the first time. 
To resolve this problem, just install missing modules with this command and replace `MODULE:MODULE` with these names:

```bash
sudo perl -MCPAN  -e 'install "MODULE:MODULE"'
```

Now, the `core` require at minimum the dependencies:

* Config::Tiny
* File::Which
* LWP::UserAgent
* Net::IP
* Text::Template
* UNIVERSAL::require
* XML::TreePP

You can check on the [agent repository](https://github.com/fusioninventory/fusioninventory-agent) to see all required dependencies. 

### Configure the agent

According with you installation, you should have a configuration file for your agent in folder `FOLDER/fusioninventory/agent.cfg`. 
Edit it and uncomment the line `#server`. Update after the url to match with your GLPI installation:

```conf
[...]
# send tasks results to a FusionInventory for GLPI server
server = http://glpi_fqdn/plugins/fusioninventory/
[...]
```

## Run the agent

Now your agent is installed, you can run it. For that, use the command:

```bash
sudo fusioninventory-agent
```

You should have this text and see what the agent do:

```bash
[info] sending prolog request to server server0
[info] running task Deploy
[info] running task ESX
[info] ESX support disabled server side.
[info] running task Inventory
[info] running task Collect
```

You can too run a network discovery, with begin IP and end IP:

```bash
# You can add --debug if needed
sudo fusioninventory-netdiscovery --first x.x.x.x --last x.x.x.x
```

You should see the devices found.

# Scheduler a task

We need import all these devices in GLPI and to do this, we need configure a task. 
But before begin, check if your agent is right in GLPI.
Go in web interface of GLPI and go in menu **Plugins => FusionInventory**. 
Go in **Général => Agents management**. 
If you have yet run your agent at least one time (see above), you should see your agent in this page.
If not, you have missed something and re-check previous steps.

## Create a task

Now, go in **Tasks => Tasks management**. 
You shound see an empty page because not have yet created a task. 
Click on button **+** beside **Tasks management** and give a name to your task (for example "discover") then click on button **Add**.

Your task is created. Don't miss to check the checkbox **Active**.

Go in tab **Jobs configuration**. Click on **Add a job**!

## Add a job

To add a job, we need define many things:

* Give it a name (for example "LAN discovery") and select a method. Here, use "network discover", then **Add**.
* Click on the name of your job recently created. You can see now you can add targets and actors.
  * The "targets" are that you will define
  * The "actors" are the agents will execute the job.
* Add a target with click on **+**, select **IP range**. Then choose the range you have previously created. 
If you haven't yet a range IP, click on the **i** below. 
This will open a new tab and you will be able to create a new IP range. 
Of course, you need to define a range your agent can see and exist! Then do **Add target**.
* For actors, add an agent you have installed and click on **Add Actor**.
* Once the 2 steps done, click on **Update**! Not go away this page before do that, otherwize your configuration will be lost!

If you go back in homepage of tasks management, you should see the task you have created.

## Start the task

When you go in **Tasks => Monitoring / logs**, you should see your task with many step names below and with zeros in front of each name. 
You can see the task is inactive for the moment. Now, the goal is to put your task in _Prepared_!

To do this, go in menu **Configuration => Task management**. 
In this page, you can see all tasks GLPI will run (core and plugins). 
Search the task named **taskscheduler** and click on it!

> In other versions of GLPI and FusionInventory, it seems the action has different name. 
Check if it is right link to FusionInventory, it should be indicated.

Does not look anything like that, but many of _us_ has problem to find this. 
One on the page of task **taskscheduler**, click on **Run**!

If you go back on your FusionInventory task, in tab **Monitoring / Logs**, it should be in **Prepared**! 
If not, you can select above the **Refresh interval** of tasks. In most cases, 5 minutes is enough.

Now, go backon your terminal and run the agent:

```
sudo fusioninventory-agent
```

And you can follow the steps of your task in GLPI. It should be passe to **Running** (in purple). 
You can click on it, this will display your agent and the job it do. 
Next, click on the name of your agent to see more details.

Depend on the IP range defined, your network speed and the number of devices to find, the task can make more or less time before finish and to pass to **Succee** (green).

## See found devices

Une fois que l'agent a fini son petit boulot, votre matériel a du être ajouté automatiquement dans GLPI. 
FusionInventory doit d'ailleurs (dans les logs) vous afficher le message suivant :

```
YYYY-DD-MM HH:MM:SS Ok  Total Found:XX Created:XX Updated:X
```

If not or if the plugin display the message `Import Denied`, it mean **Equipment import and link rules** of the plugin has too restricted.
FusionInventory has default rules in **Rules => Equipment import and link rules**. 
Your inventory can be successful, but FusionInventory not have valid them and put them in kind of blacklist in **Rules => Ignored import devices**.

If the plugin has sucessfuly imported your device, you will not find it directly in menu **Assets**. 
We need go in menu **Assets => Unmanaged devices** and surprise, all found devices are here!

If this device is right and known and should be in your inventory, you can import it, by defining the **type** of device, otherwize GLPI will refuse import it (normal because it don't know where import it). 
Once device have a _type_, check them and do **Actions** and **Import**.

Now, if you go in **Computers** menu (or else if you have imported other than computers), your devices are here with some data pre-filled. 
You can say after to GLPI to protect the data yet imported and never modify them (see that in `locks (fields)`)!

Congratulations, you have done the major part of job!

# Automate a task

Obviously, this is really nice, but run agent manually on GLPI and on Shinken, it's not very usefull... so we need automate the process. 
If you have search in all the web like me to find tutorials about this plugin, you certainly have same problem. 
In fact, there is a solution on the [official wiki](http://wiki.glpi-project.org/doku.php?id=en:config:crontab) but it's not reasy to find.

We will define 2 things:

* A `cron` task to run the GLPI pseudo-cron.
* A `cron` task for FusionInventory agent.

> In a previous version of FusionInventory, seems daemon mode not works nicely. It's why I have configured agent with `cron`. 
To run agent in daemon, add the option `-d`. You can add `--debug` to be more verbose.

## Cron task for FusionInventory

To configure agent FusionInventory with cron, we need use the `root` cron or else if you have a dedicated user, but with rights to run it. 
After, define the time to run in the cron (`sudo crontab -e`) :

```conf
# Adapt the path of fusioninventory in this job!
# The task will run all days of all months at 03h05 a.m.
5 3 * * * /usr/local/bin/fusioninventory-agent
```

Save and quit.

## Cron task for GLPI

Like said on the documentation above, we only need edit the cron of **web server user**  run GLPI! 
If you do on another user, it will not works. Be carefull, we need `php5-cli`:

```bash
sudo apt-get install php5-cli
```

One done, edit the cron of user:

```bash
sudo crontab -u www-data -e
```

And add the line at the end of file:

```conf
# The GLPI path need match path of your installation!
*/1 * * * * /usr/bin/php5 /var/www/glpi/front/cron.php &>/dev/null
```

Save and quit the **crontab**. 

This task will execute the file `cron.php` each minute. 
You can define an another frequency according your installation, see what is the best for you. 
The job will create an activity in GLPI and execute the automatic crontasks of GLPI, included our `taskscheduler`.

# Conclusion

If you are patient, You can wait 3h00 a.m. to see if it works... else you can modify time execution of the agent to test your process. 
You can follow after your inventory in GLPI in **Monitoring / Logs** of Fusioninventory menu. 
It will go in mode _Prepared_, and after go step to step and finish. 
You can too check the right working jobs with logs of the Cron:

```bash
grep CRON /var/log/syslog
```

Now, you have a solution to automate the inventory of your assets easily and so have the data up to date.


