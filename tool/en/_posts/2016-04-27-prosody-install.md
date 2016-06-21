---
layout: post
title: Install Prosody
lang: en
ref: prosody
modified:
description: How to install Prosody
tags: [tutoriel, prosody]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments:
share:
date: 2016-04-27T14:15:05+01:00
---

# Introduction

This tutorial will help you to install an instant messaging server with [Prosody](https://prosody.im/). Prosody is a communication server using the protocol [XMPP](https://en.wikipedia.org/wiki/XMPP) which allows for instant messaging either for you or for your business, it is usable on all terminal types. Using a standard protocol let use it with any XMPP client.

> **Info :**  a list of XMPP clients more or less up to date is available on [Wikipedia](https://fr.wikipedia.org/wiki/Liste_de_clients_XMPP). Personally, I use [Gajim](https://gajim.org/index.fr.html) on Linux and [Jitsi](https://jitsi.org/) on Windows.

# Prerequisite

You will only need a Linux server with a `root` access. I will also explain how to link accounts to Active Directory.

# Installation of Prosody

## Method 1

To install Prosody, you can do it via the packages :

```bash
sudo apt-get update
sudo apt-get install prosody
```

Ubuntu Server ( 14.04.4 LTS ), the deposit is up to date and offers the latest version.

## Method 2

You can also add the repos of Prosody which will be easier to maintain :

```bash
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add -
sudo apt-get update
sudo apt-get install prosody
```

# Configuration

You will have now set Prosody. The first thing to do is create a _VirtualHost_ in a configuration file. Prosody configuration files are ended with `.lua`.

Go to the Prosody directory and look at what is in folders :

```bash
cd /etc/prosody
ls -l
```

You should have the following folders:

* `certs` that will contains your certificates and keys.
* `conf.d` which will be active configurations (much like Apache or Nginx system) using symbolic links.
* `conf.avail` : available configuration files.

You will now create a configuration file.

Replace `mail.fr` by the name of your domain to match your configuration. In this tutorial, we'll use email, so users log in with their email instead of their Active Directory login. Also it displays the status of the people in Outlook (if you have Jitsi). So I'll make it with mail for this tutorial.

```bash
# If folder does not exist
sudo mkdir conf.avail
# Create configuration file
sudo vi conf.avail/mail.fr.cfg.lua
```

Then copy the following lines :

```lua
--- Virtual hostname ---
VirtualHost "mail.fr"

--- Components ---
Component "salons.mail.fr" "muc"
    name = "Room server for mail.fr"
    restrict_room_creation = "admin"
```

Your virtual hostname will be the address to use to connect : eg ` user@mail.fr`.

You can also disable the host by adding the line `enabled = false` after.

The `Components` are modules that you can add to the prosody server. In this case, a `MUC` is defined ("Multi-user Chat" : in short, a "room"), with corresponding addresses (`salons.mail.fr`).

Putting `admin` for `restrict_room_creation` only allows administrators to create **persistent room** ! This means that other users can only create temporary conferences. Most XMPP client then propose administrators to configure the rooms as they wish. If you do not want this feature, simply delete the line.

# Authentication

Here's how to set an administrator for Prosody. Open the main configuration file (`sudo vi prosody.cfg.lua`) and add your user. You can also take the opportunity to browse the file. It is fairly well explained.

```lua
admins = { "admin@mail.fr" }
```

Prosody can manage authentication with your AD server. As said above, we will ensure that people can connect with their email. To achieve this, it is necessary that email is obviously set in your AD for each user.

To manage authentication, we need [ Cyrus ] ( https://cyrusimap.org/mediawiki/index.php/Cyrus_SASL ). Here's how to install it :

```bash
sudo apt-get install lua-cyrussasl sasl2-bin
```

Then we will indicate this in our configuration ( vi `conf.avail/mail.fr.cfg.lua`). Now your file should look like this :

```lua
--- Nom de l'hôte virtuel ---
VirtualHost "mail.fr"

modules_enabled = {
    "saslauth";
    }

allow_registration = false;

c2s_require_encryption = true
s2s_secure_auth = true

authentication = "cyrus"

allow_unencrypted_plain_auth = true
[...]
```


You must then edit the **saslauthd** configuration file ( `sudo vi /etc/default/saslauthd`) :

```conf
[...]
START=yes
[...]
MECHANISMS="ldap"
[...]
# Here we set the ldap option file
MECH_OPTIONS="/etc/saslauthd.conf"
```

Now, define the options with which saslauthd will connect to your AD . Add and edit the file `/ etc / saslauthd.conf` :


```conf
ldap_servers: ldap://ip_server
ldap_search_base: dc=domain,dc=com

ldap_bind_dn: DOMAIN\user
ldap_bind_pw: password
ldap_start_tls: no
ldap_auth_method: bind

# Filter below is for connections with username
# ldap_filter: (samAccountName=%U)
# Here we use mail instead :
ldap_filter: (mail=%U@mail.fr)
```

Replace the various data with your own configuration, save and exit.


### Test authentication

You must restart the service saslauthd :

```bash
sudo service saslauthd restart
```

Now you can test the setup with the following command :

```bash
# In case the user has an email "u.user@mail.fr"
sudo testsaslauthd -u u.user -p password
```

If you have the following output : `0: OK "Success"` your authentication works. Otherwise, check out your options for saslauthd and if you have installed all dependencies.

## Prosody and Sasl

So that the user has access to the prosody saslauthd socket, we take care to add it to sasl group :

```bash
sudo gpasswd -a prosody sasl
```

You can verify that the user has access to prosody saslauthd :

```bash
sudo su prosody -s /bin/bash
/usr/sbin/testsaslauthd -u u.user -p password
```

Finally, we must add one last configuration file in `/etc/sasl/`. The folder probably does not exist, you will need to create :

```bash
cd /etc/
sudo mkdir sasl
# Then edit the following file
sudo vi /etc/sasl/prosody.conf
```

Add the this two lines :

```conf
pwcheck_method: saslauthd
mech_list: PLAIN
```

Save and quit.

Restart saslauthd and prosody service :

```bash
sudo service saslauthd restart
sudo service prosody restart
```

# Test a client

You will now be able to test with a client (Gajim, Jitsi, Pidgin...). You simply enter :

* IP XMPP server, leaving port 5222.
* Your credentials : email and password
* Accept the certificate

In most clients you can configure a proxy if necessary.

Normally, the connection is expected to pass and your status should be **connected** ! If this is not the case, look at your logs :

```bash
sudo tail -f /var/log/prosody/prosody.log
# or
sudo tail -f /var/log/prosody/prosody.err
```

The files of logs can be defined in the `prosody.cfg.lua` file.

# Manage Certificates

With Prosody, you can define certificate (self-signed or not) for each **VirtualHost**. In the main configuration file (`prosody.cfg.lua`), you should have the following lines :

```lua
-- These are the SSL/TLS-related settings. If you don't want
-- to use SSL/TLS, you may comment or remove this
ssl = {
    key = "/etc/prosody/certs/localhost.key";
    certificate = "/etc/prosody/certs/localhost.crt";
}
```

These lines indicate the path to the key and certificate to the **localhost** server and Prosody server. It is therefore defined two times. As you can see, the **localhost** server is disabled ( `enabled = false`). So it should not bother us.

By cons, for your new virtual server (`mail.fr`), it would be redefined.

The goal here is to have a certificate authority (`localhost`) and therefore our sign other certificates.

## Install Lua-expat 1.3

If you have a version problem for lua-expat, you must add the repository universe of **vivid** (`sudo vi /etc/apt/sources.list`) :

```conf
deb http://us.archive.ubuntu.com/ubuntu vivid main universe
```

Then updates the deposits and install lua-expat :

```bash
sudo apt-get update
sudo apt-get install lua-expat
```

> **Note :** during the update packages, it is possible that you have errors. Install **lua-expat** and then you can comment out the line added in `sources.list`.

## Generate a request file

Now in order to generate our ssl certificate for our virtual host **mail.fr**, you must have a request file :

```bash
sudo prosodyctl cert request mail.fr
```

Type `Enter` to validate data by default or enter new information. It should look something like this :

```bash
Choose key size (2048):
Generating RSA private key, 2048 bit long modulus
....................................+++
..................................................+++
e is 65537 (0x10001)
Key written to /var/lib/prosody/mail.fr.key
Please provide details to include in the certificate config file.
Leave the field empty to use the default value or '.' to exclude the field.
countryName (FR):
localityName (The Internet):
organizationName (Your Organisation): MyBusiness
organizationalUnitName (XMPP Department): DSI
commonName (mail.fr):
emailAddress (xmpp@mail.fr): m.mail@mail.fr

Config written to /var/lib/prosody/mail.fr.cnf
Certificate request written to /var/lib/prosody/mail.fr.req
```

## Sign with the CA

The certificate authority will now help us to create and sign our key :

```bash
sudo openssl x509 -req -days 730 -in /var/lib/prosody/mail.fr.req -CA /etc/prosody/certs/localhost.crt -CAkey /etc/prosody/certs/localhost.key -set_serial 01 -out /var/lib/prosody/mail.fr.crt
```

You should have the following output :

```bash
Signature ok
subject=/C=FR/L=The Internet/O=MyBusiness/OU=DSI/CN=mail.fr/emailAddress=m.mail@mail.fr
Getting CA Private Key
```

## Add Certificate

Now that we have a lovely self-signed certificate, we can add it to our VirtualHost. Open the configuration file (`sudo vi conf.avail/mail.fr.cfg.lua`) and add it as follows :

```lua
ssl = {
        key = "/var/lib/prosody/mail.fr.key";
        certificate = "/var/lib/prosody/mail.fr.crt";
}
```

> **Note :** be careful to add these lines after the declaration of `VirtualHost`. If you add them before, it will not work.

Now you can restart prosody :

```bash
sudo service prosody restart
```

During the reconnection of your client, you should have a new acceptance of the certificate request. You can display it to see the data you enter above.

Confirm and agree . Check the box **ignore** if you do not want to have this warning.

You can then repeat these operations if you have other virtual hosts to add. Well sign your certificates with the same authority (localhost in this case) !

# Other possibilities

Prosody allows for many things, like create persistent rooms, display welcome messages , receive files etc ...

## The Rooms

For the creation of rooms, you normally have written the following lines at the beginning of the tutorial :

```conf
--- Components ---
Component "salons.mail.fr" "muc"
    name = "Serveur de Salons pour mail.fr"
    restrict_room_creation = "admin"
```

This means that the creation of rooms will be possible only by administrators. Moreover, the rooms are created by the client !

You will have the option in your client to define whether the show is permanent, open, protected by a password,... Other people connected can then search for available rooms in their client and connect if they have the sufficient rights.

## Word of the Day

You can set a word of day, adding the following line (in `prosody.cfg.lua`) :

```lua
motd_text = [[Welcome on Instant messaging server.]]
```

Restart prosody to apply the change.

## Plugins Paths

If you take the urge to add more plugins, you can set the path thereof through the following :

```lua
plugin_paths = { "/usr/lib/prosody/modules", "/usr/lib/prosody/prosody-modules"}
```

Restart prosody to apply the change.

# Conclusion

In the end, Prosody is very comprehensive. There are obviously other XMPP server but I find that Prosody provides everything you need for instant messaging and allows you to configure your connections accurately.
