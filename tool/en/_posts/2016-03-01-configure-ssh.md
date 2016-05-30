---
layout: post
title: Install and configure SSH
lang: en
ref: ssh
modified:
description: Eg ssh configuration
tags: [ssh]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-03-01T16:40:05+01:00
---

# Introduction

This article describes how to set up an ssh connection and the corresponding configuration files. When has to manage several Linux servers, it is essential to use this type of connection and know how to configure. This article is not exhaustive and I would certainly have to add new things later.

## Prerequisites

To make connection tests, it is best to have the following prerequisites:

* 1 host machine (Desktop or Server) and at least one Linux server. In my case, I use Ubuntu Desktop and Ubuntu Server. But other distributions may very well do the trick.
* Access **root** on both machines.

# SSH what is it?

**Secure Shell** is both a computer program and a secure communications protocol. This protocol works with a cryptographic key system which subsequently ensures that all TCP segments are authenticated and encrypted.

# Install SSH

Under Linux, it is very easy to install an ssh server and a client. On Windows, the best known SSH client is [Putty] (http://www.putty.org/) which is very well done and easily configurable. I will only explain the Linux section in this article.

To install ssh type the following command:

```bash
sudo apt-get update
sudo apt-get install openssh-server
```

Once the package installed, you can verify the presence of the server with the following command :

```bash
ssh
```

This will display following lines :

```bash
usage: ssh [-1246AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-L [bind_address:]port:host:hostport] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port]
           [-Q cipher | cipher-auth | mac | kex | key]
           [-R [bind_address:]port:host:hostport] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] [user@]hostname [command]
```

As you see , this is not the options that are missing.

# Connect SSH

Assume we have the following architecture :

|  Hostname     |  Role   |  Private FQDN             |
|:-------------:|:-------:|:-------------------------:|
| ubuntudesktop | Client  | ubuntudesktop.localdomain |
| ubuntuserver  | Serveur | ubuntuserver.localdomain  |

You must already have a user and password on server (or at least you have this information).

Then type the following command on the Linux Client replacing my name with the name of the **account server** you are trying to connect :

```bash
ssh algorys@ubuntuserver.localdomain
```

> **Note :** You may well replace the FQDN of the server IP. That will work, but it will be less clean for later when you need to change IP or server.

Your console should display the following :

```bash
The authenticity of host 'ubuntuserver.localdomain (192.168.10.220)' can't be established.
ECDSA key fingerprint is SHA256:ALQBuTz6WppLa5vdXJU4t+JUvBwkhTfb61j7rfPgi38.
Are you sure you want to continue connecting (yes/no)?
```

This indicates that the authenticity of the server is unknown and tells you the corresponding IP. This is normal, you can type ` yes` to complete the operation.

The terminal will ask then the user's password (the server) :

```bash
algorys@ubuntuserver's password:
```

Enter the password and type `Enter` .

If all is well you should connect to the remote server. You may also notice that the terminal hostname has changed.

# Private Key and Public Key

You managed to establish your first connection by ssh but finally it's similar to open a terminal on this machine, enter the user name and password. So this is currently not very useful.

The goal now is to use a pair of keys to avoid having to enter your password and your username every time you log. And especially to properly encrypt your connection.

To generate a pair of keys, enter the following command on your client :

```bash
cd ~
ssh-keygen
```

You will have several steps to validate :

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/algorys/.ssh/id_rsa):
Created directory '/home/algorys/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/algorys/.ssh/id_rsa.
Your public key has been saved in /home/algorys/.ssh/id_rsa.pub.
The key fingerprint is:
d7:0b:69:7f:5b:3e:70:25:9a:a1:8f:66:55:3d:32:3f algorys@ubuntuserver.localdomain
```

If you have observed the steps carefully, you could see that _ssh-keygen_ generated two file in a folder `.ssh` (`ll .ssh/`) :

* A **id_rsa** file : it is your private key. Never communicate to another server or person ! This is what will allow to decrypt your public key.
* A **id_rsa.pub** file : this is your public key. This is one you're going to give to your remote server.


Now that you have a key pair, you must give your public key to the server you want to connect via SSH.

## Allow key

To allow a key on your server, create a specific file in the .ssh folder of the affected user on the server. In this example, I'm using the same username on the client and on the server. But it is quite possible to use two different names.

Display your public key on your client :

Cat your public key on your guest :

```bash
cat ~/.ssh/id_rsa.pub
```

It should look like this :

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBSG/JUGFA3NP+9vSTG/6pnXnzRvSLCZsG+M2+a0IAUrGSILLdlSy/XswYxAbXEp4gG4N7TM8opTPozFwcWn0ubjzSsZLtlcJmfWaQE0C4FTqdKKxJQEENyVlFB8V1k125ZQWKQtoLbOAuQTKd+yedj1ijnU9GcJvMOydqijRZeSIyjSaJn4vV6qvzrTfS7g5USalGtQ+rPW/nS0i6Dp4yxraOCuhVMrcpm0UO6togrf862gtBTqntSAyYREhtvguYFOE95m/mwZim4rkOq6AFSFTZwaEMUu1rTIr467ONG9zekpxVH1fWfA8IO+X5wrAET2sNqHw7SiwZsQ9bFE/B algorys@ubuntudesktop.localdomain
```

On the server type the following command (as the desired user) :

```bash
cd ~
mkdir .ssh
vi .ssh/authorized_keys
```

Paste all your public key (of _ssh-rsa_ to _user@domain_ ) in this file. Save and exit.

Now you have configured your SSH key on the server. Then just test the SSH command to see if everything works :

```bash
ssh algorys@ubuntuserver.localdomain
```

Normally the connection will be no password request. If the server asks you a password, you have missed a step.

# Preset your SSH access

Sometimes it can be quite long to type the full FQDN of your server or remember all the IP of your servers. To avoid this and especially to connect faster when working on many servers, you can create a configuration file in your SSH folder.

Your configuration file must be located in `~/.ssh/config`. You can then configure the file as follows (This is just example, you have to adapt hostanme and IP) :

```conf
Host wiki
  HostName monwiki.localdomain

Host puppet
  HostName 192.168.10.220

Host *
  ControlMaster auto
  ControlPath ~/.ssh/sockets/%r@%h-%p
  ControlPersist 600
  UseRoaming no
```

The first two **Host** `wiki` and `puppet`, configure two servers, one with an IP and the other with its domain name.

The last configuration allows you to temporarily keep your ssh connection (in fact the socket) for 600 seconds. This prevents delays when you need to connect, reconnect, several times in a short time. This comes in handy, especially for connections with sources of management servers (such as Gitlab or Github), which often requires SSH access when pushed branches. This configuration is optional.

Now you can run the following commands to connect to your favorite servers :

```bash
ssh wiki
```

```bash
ssh puppet
```

This is much faster right ?

# Conclusion

Mount a SSH connection with OpenSSH is very simple and allows to gain speed during the administration of your servers. It is also very easy to give names to your connections, thanks to the `config` file.


