---
layout: post
title: Install Nagios Plugins and SNMP scripts
lang: en
ref: nagios-plugins-snmp
modified:
description: How to install Nagios Plugins and SNMP scripts
tags: [tutoriel, snmp, monitoring]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-12-21T11:20:00+01:00
---

# Introduction

In this tutorial, we'll install [Nagios Plugins](https://www.nagios.org/projects/nagios-plugins/) to get scripts who help to monitor hosts or services. We also install [Nagios SNMP from Manubulon](http://nagios.manubulon.com/) who give general snmp scripts.

This kind of scripts is very useful when you want to monitor your infrastructure. This list is obviously not exhaustive and many other scripts are available on the internet. Links will be available at the end of this tutorial.

# Prerequisites

The more practice in general is to install those scripts on the server you use to monitor (usually a Shinken server, Nagios or more recently Alignak).

For scripts using the [snmp protocol](https://fr.wikipedia.org/wiki/Simple_Network_Management_Protocol), you must enable it on the hosts or devices you want to monitor. (This can be PC, Servers, Printers, Switch).

# Enable SNMP

On many devices you have a configuration section to enable SNMP services. For example, on printers, you'll have a web interface who let you configure your printer. Just find the right section or refer to your manual. For switches, this is usually about the same way. On others, you will need to go through command lines.

Below, you will find an example for Linux and Windows.

## On Linux Server

This example is for Debian distributions, but it can normally be easily adapted on any linux distribution.

First, to get snmp service, install **snmpd** package:

```bash
sudo apt-get install snmpd
```

The snmpd service should now be running:

```bash
user@server:$ sudo services snmpd status
snmpd.service - LSB: SNMP agents
   Loaded: loaded (/etc/init.d/snmpd; bad; vendor preset: enabled)
   Active: active (running) since Wed 2016-12-21 02:55:58 PST; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 17135 ExecStop=/etc/init.d/snmpd stop (code=exited, status=0/SUCCESS)
  Process: 17144 ExecStart=/etc/init.d/snmpd start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/snmpd.service
           └─17153 /usr/sbin/snmpd -Lsd -Lf /dev/null -u snmp -g snmp -I -smux mteTrigger mteTriggerConf -p /run/snmpd.pid

Dec 21 02:55:58 alignak systemd[1]: Starting LSB: SNMP agents...
Dec 21 02:55:58 alignak snmpd[17144]:  * Starting SNMP services:
Dec 21 02:55:58 alignak snmpd[17151]: /etc/snmp/snmpd.conf: line 145: Warning: Unknown token: defaultMonitors.
Dec 21 02:55:58 alignak snmpd[17151]: /etc/snmp/snmpd.conf: line 147: Warning: Unknown token: linkUpDownNotifications.
Dec 21 02:55:58 alignak snmpd[17151]: Turning on AgentX master support.
Dec 21 02:55:58 alignak systemd[1]: Started LSB: SNMP agents.
Dec 21 02:55:58 alignak snmpd[17153]: NET-SNMP version 5.7.3
```

As you can see **snmpd** daemon is verbose and you see if there is any problem in your configuration.

Here I had some warnings. This king of warning is not necessarily serious but you must pay attention to this kind of alert. By doing a quick search on the internet, I saw that it was just an oversight of a previous version, so I've just comment this lines.

You must now set up your service in `/etc/snmp/snmpd.conf` which defines your SNMP community. By default community is named **public** (for IPv4 and IPv6).

**WARNING:** I strongly advise you to change this when you are in production !

Below I call my community _mycommunity_ just for example:

```conf
#rocommunity public  localhost
                                                 #  Default access to basic system info
 rocommunity mycommunity  default    -V systemonly
                                                 #  rocommunity6 is for IPv6
 rocommunity6 mycommunity  default   -V systemonly
[...]
sysLocation    Server Alignak
sysContact     my_email@email.com
```

Don't forget to restart service after:

```bash
sudo service snmpd restart
```

You can also install the `snmp` package to have some command who can test your snmp configuration.

```bash
sudo apt-get install snmp
# Test on local loop
snmpwalk -v1 127.0.0.1 -c mycommunity
```

There is many other possible configurations (and tutorials) that you can find on the web.

## On Windows Server

Usually I'm not too comfortable with Powershell, but for once I admit that it simplifies things. Then it avoids opening Server Manager. So open a Powershell console with Administrator rights and type the following commands:

**Note:** The following configuration is under Windows Server 2012.

```powershell
PS C:\Windows\system32> Get-WindowsFeature -Name *snmp* | Install-WindowsFeature

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Fournisseur WMI SNMP}
```

It tells us that the operation was successful and that it is not necessary to restart the server (Ouf !). You can also simply check if all is installed:

```powershell
PS C:\Windows\system32>  Get-WindowsFeature -Name *snmp*

Display Name                                            Name                       Install State
------------                                            ----                       -------------
        [X] Outils SNMP                                 RSAT-SNMP                      Installed
[X] Service SNMP                                        SNMP-Service                   Installed
    [X] Fournisseur WMI SNMP                            SNMP-WMI-Provider              Installed
```

Once done, we have to configure it like in Linux, so type `.\services.msc` in Powershell to open **Services** windows.

Now search for `SNMP services`, right click on and choose `Properties`. You normally have new tabs (Compared with other services), like _Agent_ and _Security_ ! If you do not have this tabs, just restart service: with right-click on _SNMP services_ and _Restart_ or with Powershell: _Restart-Services SNMP_). This should solve the problem.

### Security Tab

Here you can define your community. Normally, by default, you have public defined. I **strongly** advise you to change this when you are in production ! You can also define who can access to this community. Just click on **Add** or **Modify** and type the appropriate IPs or FQDNs.

### Agent Tab

Here you can define SNMP contact and place of the server, as well as the services he manage. In general, I let default options...

Now your SNMP is configured. Unfortunately, I have not found a Powershell command to test the SNMP configuration. But we can do it from a Linux server:

```bash
snmpwalk -v1 <ip_server> -c mycommunity
```

# Nagios Plugins

Nagios commands are usefull to check services of an host locally. The official Plugins are the base, but there are thousands of them found on the internet. Here is how to install Officials:

**Note:** You must be under Linux system to install this plugins.

```bash
# To get ordered, I make the following folders:
cd ~; mkdir tools; cd tools
wget https://nagios-plugins.org/download/nagios-plugins-2.1.4.tar.gz#_ga=1.188074479.692094193.1469016700
tar xzvf nagios-plugins-2.1.4.tar.gz
# Now install it
cd nagios-plugins-2.1.4/
# Here I give the rights to "alignak" user, because it was installed on an Alignak server. You can give rights to another user, if you want.
# You can also define "prefix" install with "--prefix=<path>" for example.
./configure --with-nagios-user=alignak --with-nagios-group=alignak
make
sudo make install
# Remove archive
cd ..; rm nagios-plugins-2.1.4.tar.gz
```

Now if you check your install (by default in `/usr/local/nagios/`) you will find a folder called `libexec` who contains all plugins ! Most followthe syntax `check_<service>`. For example, the plugin SMTP is called `check_smtp`.

All of this plugin have generally an help command, callable by `-h`:

```bash
user@server:$ ./check_dns -h
[...]
Usage:
check_dns -H host [-s server] [-q type ] [-a expected-address] [-A] [-n] [-t timeout] [-w warn] [-c crit]
[...]
```

This will tell you how to use the plugin. Take time to carefully look at each plugin and their syntax. If in the future you install or download other plugins, I advise you to install them in this same directory (or not very far) in order not to have scripts that roam everywhere.

# Nagios SNMP from Manubulon

This other plugins are very usefull to get data from your servers. There are not many but all are very useful.

To get this plugins, just download the archive on website : [http://nagios.manubulon.com/](http://nagios.manubulon.com/).

```bash
cd ~/tools
wget http://nagios.manubulon.com/nagios-snmp-plugins.1.1.1.tgz
tar xzvf nagios-snmp-plugins.1.1.1.tgz
cd nagios_plugins/
```

Before going any further, you may notice that all these plugins use **perl**. In the provided README in archive, you can read that some perl modules are required, as well as the _utils.pm_ file.

Normally, if you have installed Nagios Plugins before, you have this file in the right location (_/usr/local/nagios/libexec/utils.pm_).

Check also that user nagios (that you have defined before) can write in _/tmp/_ and your perl bin is located in /usr/bin (do `which perl` to know if it's good).

## Install perl dependencies

You can install this dependencies with your distribution packages or use `cpan` or `cpanm`. I prefer _cpanm_ because you can easily manage your perl dependencies after.

```bash
sudo apt-get install cpanminus
```

Now install required modules:

```bash
sudo cpanm Net::SNMP
sudo cpanm Getopt::Long
# You can also install without sudo, but it'll install in your home. Plugins will search in _/usr/local_.
```

## Install scripts

Now that you have all the requirements, launch install script. He will check your configuration and you'll may be able to change folders to suit your installation:

```bash
user@server:$ sudo ./install.sh
###### Nagios snmp scripts installer ######

Will install all script(s)

What is your perl location ? [/usr/bin/perl] 
Net::SNMP module version is v6.0.1 [OK]
Module Getopt::Long found [OK]

What is your nagios plugin location ? 
Note : file utils.pm must be present in it [/usr/local/nagios/libexec] 

Where do you want the plugins to put temporary data (only used by some plugins) ? 
Nagios user must be able to write files in it [/tmp] 

Will now install all script(s) : 
in directory    : /usr/local/nagios/libexec
perl            : /usr/bin/perl
temp directory  : /tmp

OK ? [Y/n]

Installing check_snmp_boostedge.pl : OK

Installing check_snmp_css.pl : OK

Installing check_snmp_linkproof_nhr.pl : OK

Installing check_snmp_nsbox.pl : OK

Installing check_snmp_vrrp.pl : OK

Installing check_snmp_cpfw.pl : OK

Installing check_snmp_env.pl : OK

Installing check_snmp_load.pl : OK

Installing check_snmp_process.pl : OK

Installing check_snmp_win.pl : OK

Installing check_snmp_css_main.pl : OK

Installing check_snmp_int.pl : OK

Installing check_snmp_mem.pl : OK

Installing check_snmp_storage.pl : OK

Installation completed OK
You can delete all the source files and directory
Remember to look for informtation at http://www.manubulon.com/nagios/
```

You can now see the new plugins installed in your nagios folder (here _/usr/local/nagios/libexec/_). As you can see, script follow this syntax: `check_snmp_<service>`.

# Using SNMP scripts and others

Most of these plugins have common commands. In general, you must specify:

* The target host with `-H <host_ip>`
* The warning level with `-w` follow by an integer or a percentage
* The critical level with `-c` follow by an integer or a percentage
* The community with `-C` follow by community name

In some cases, you will also need to specify the SNMP version, the SNMP port, the output data (for _perfdata_ readers) and then specific values depending on the service you want to monitor.

# Conclusion

This tutorial obviously does not cover all the possibilities offered by the Nagios plugins. But it already gives you a solid foundation to start with.

You can also find others commands and tutorials on the following links:

* [Alignak-checks-snmp](https://github.com/Alignak-monitoring-contrib/alignak-checks-snmp): provides checks pack for monitoring hosts with SNMP
* [Exchange Nagios Plugins](https://exchange.nagios.org/directory/Plugins): thousands of other plugins
* [Monitoring Configurations](https://github.com/mohierf/Monitoring-configuration): repos who contains many configurations help for monitoring.

And there are surely plenty of other...
