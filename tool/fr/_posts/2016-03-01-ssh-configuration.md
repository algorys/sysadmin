---
layout: post
title: Installer et configurer SSH
lang: fr
ref: ssh
modified:
description: Exemple de configuration pour ssh
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

Cet article décrit comment configurer une connexion ssh, ainsi que les fichiers de configuration correspondant. Lorsqu'on est amené à gérer plusieurs serveurs Linux, il est indispensable de pouvoir utiliser ce type de connexion et de savoir les configurer. Cet article n'est pas exhaustif et je serais certainement amené à rajouter de nouvelles choses par la suite.

## Prérequis

Afin de faire des tests de connexion, il est préférable d'avoir les prérequis suivants :

* 1 Machine hôte (Desktop ou Server) et au moins un serveur Linux. Dans mon cas, j'utilise un Ubuntu Desktop et un Ubuntu Server. Mais les autres distributions peuvent très bien faire l'affaire.
* Un accès **root** sur ces deux machines.

# Le SSH c'est quoi ?

**Secure Shell** de son vrai nom est à la fois un programme informatique et un protocole de communication sécurisé. Le protocole fonctionne avec un système de clés de chiffrement qui par la suite assure que tous les segments TCP sont authentifiés et chiffrés.

# Installer SSH

Sous Linux il est très simple d'installer un serveur ssh, ainsi qu'un client. Sous Windows, le client SSH le plus connu est [Putty](http://www.putty.org/) qui est très bien fait et facilement configurable. Je ne couvrirais que la partie Linux dans cet article.

Pour installer ssh tapez la commande suivante :

```bash
sudo apt-get update
sudo apt-get install openssh-server
```

Une fois le paquet installé, vous pouvez vérifier la présence du serveur avec la commande suivante :

```bash
ssh
```

Cela devrait vous afficher les lignes suivantes :

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

Comme vous le voyez, ce n'est pas les options qui manquent.

# Se connecter en SSH

Partons du principe que nous avons l'architecture suivante :

| Nom de l'hôte | Rôle    |  FQDN Privé               |
|:-------------:|:-------:|:-------------------------:|
| ubuntudesktop | Client  | ubuntudesktop.localdomain |
| ubuntuserver  | Serveur | ubuntuserver.localdomain  |

Pour vous connecter en ssh sur un serveur, il faut que vous ayez déjà un utilisateur et un mot de passe sur celui-ci (ou tout du moins que vous ayez ces informations). 

Ensuite tapez la commande suivante sur le Linux Client en remplaçant mon nom par celui du **compte du serveur** auquel vous essayez de vous connecter :

```bash
ssh algorys@ubuntuserver.localdomain
```

> **Note :** Vous pouvez très bien remplacer le FQDN par l'IP du serveur. cela fonctionnera, mais ce sera moins propre pour la suite lorsque vous devrez changer d'IP ou de serveur.

Votre console devrait afficher le texte suivant :

```bash
The authenticity of host 'ubuntuserver.localdomain (192.168.10.220)' can't be established.
ECDSA key fingerprint is SHA256:ALQBuTz6WppLa5vdXJU4t+JUvBwkhTfb61j7rfPgi38.
Are you sure you want to continue connecting (yes/no)?
```

Cela indique que l'authenticité du serveur n'est pas connue et vous précise l'IP correspondante. C'est normal, vous pouvez taper `yes` pour valider l'opération.

Le terminal vous demandera ensuite le mot de passe de l'utilisateur (du serveur) :

```bash
algorys@ubuntuserver's password: 
```

Renseignez le mot de passe et tapez `Entrée`.

Si tout est correct vous devriez vous connecter sur le serveur distant. Vous pouvez d'ailleurs remarquer que le nom de machine du terminal a changé.

# Clé Privée et Clé Publique

Vous avez réussi à établir votre première connexion en ssh mais du coup cela revient un peut à ouvrir un terminal sur cette machine, entrer le nom de l'utilisateur et le mot de passe. Ce n'est donc pour le moment pas très utile. 

Le but maintenant est d'utiliser une paire de clés afin justement d'éviter d'avoir à saisir votre mot de passe et votre identifiant à chaque fois que vous vous logger. Et surtout de chiffrer correctement votre connexion.

Pour générer une paire de Clés, entrez la commande suivante sur votre client :

```bash
cd ~
ssh-keygen
```

Vous allez avoir plusieurs étapes à valider :

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/algorys/.ssh/id_rsa):          # Choisissez dans quel dossier vous voulez sauvegarder les clés.
Created directory '/home/algorys/.ssh'.
Enter passphrase (empty for no passphrase):                               # Entrez une "passphrase". Cette étape est optionnelle.
Enter same passphrase again: 
Your identification has been saved in /home/algorys/.ssh/id_rsa.
Your public key has been saved in /home/algorys/.ssh/id_rsa.pub.
The key fingerprint is:
d7:0b:69:7f:5b:3e:70:25:9a:a1:8f:66:55:3d:32:3f algorys@ubuntuserver.localdomain
```

Si vous avez observé attentivement les étapes, vous avez pu voir que _ssh-keygen_ a généré deux fichier dans un dossier `.ssh` (`ll .ssh/`) :

* Un fichier **id_rsa** : c'est votre clé **privée**. Ne la communiquer jamais à un autre serveur ou personne ! C'est ce qui permettra de déchiffrer votre clé publique.
* Un fichier **id_rsa.pub** : c'est votre clé **publique**. C'est elle que vous allez pouvoir donner à votre serveur distant. 

Maintenant que vous avez une paire de clés, il va falloir allez donner votre clé publique au serveur auquel vous souhaitez vous connecter en SSH.

## Autoriser une clé

Pour autoriser une clé sur votre serveur, il faut créer un fichier spécifique dans le dossier .ssh de l'utilisateur concerné sur le serveur. Dans cet exemple j'utilise le même nom d'utilisateur sur le client et sur le serveur. Mais il est tout à fait possible d'utiliser 2 noms différents.

Affichez votre clé publique sur votre client :

```bash
cat ~/.ssh/id_rsa.pub
```

Votre clé devrait ressembler à ça :

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBSG/JUGFA3NP+9vSTG/6pnXnzRvSLCZsG+M2+a0IAUrGSILLdlSy/XswYxAbXEp4gG4N7TM8opTPozFwcWn0ubjzSsZLtlcJmfWaQE0C4FTqdKKxJQEENyVlFB8V1k125ZQWKQtoLbOAuQTKd+yedj1ijnU9GcJvMOydqijRZeSIyjSaJn4vV6qvzrTfS7g5USalGtQ+rPW/nS0i6Dp4yxraOCuhVMrcpm0UO6togrf862gtBTqntSAyYREhtvguYFOE95m/mwZim4rkOq6AFSFTZwaEMUu1rTIr467ONG9zekpxVH1fWfA8IO+X5wrAET2sNqHw7SiwZsQ9bFE/B algorys@ubuntudesktop.localdomain
```

Sur le serveur tapez la commande suivante (en tant que l'utilisateur souhaité) :

```bash
cd ~
mkdir .ssh
vi .ssh/authorized_keys
```

Et coller **toute** votre clé publique (de _ssh-rsa_ à _utilisateur@domain_) dans ce fichier. Sauvegardez et quittez.

Voilà, vous avez configuré votre clé SSH sur le serveur. Il ne vous reste plus qu'à tester la commande SSH pour voir si tout fonctionne :

```bash
ssh algorys@ubuntuserver.localdomain
```

Normalement la connexion se fera sans demande de mot de passe. Si le serveur vous redemande un mot de passe, vous avez du rater une étape.

# Pré-Configurer mes accès SSH

Parfois, il peut être assez long de tapez le FQDN complet de votre serveur ou de vous souvenir de toutes les IP de vos serveurs. Pour éviter cela et surtout pour vous connecter plus rapidement lors d'une intervention sur l'un de vos serveur, vous pouvez créer un fichier de configuration dans votre dossier SSH (Sur votre client donc).

Votre fichier de configuration doit se trouver dans `~/.ssh/config`. Vous pouvez ensuite configurer le fichier comme suit :

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

Les deux premiers **Host**, `wiki` et `puppet`, configurent deux serveurs, l'un avec une IP et l'autre avec son nom de domaine.

La dernière configuration permet de garder temporairement votre connexion ssh (ou plus préciséement le socket de celle-ci) pendant 600 secondes. Cela évite les lenteurs lorsque vous devez vous connecter, reconnecter, plusieurs fois en peu de temps. Cela est très pratique, notamment pour des connections avec des serveurs de Gestion de Sources (comme Gitlab ou Github) qui demande souvent des accès SSH lors de poussée vers les branches. Cette configuration est donc facultative.

Maintenant vous pouvez exécuter les commandes suivantes pour vous connecter à vos serveurs préférés :

```bash
ssh wiki
```

```bash
ssh puppet
```

C'est beaucoup plus rapide non ?

# Conclusion

Monter une connexion SSH avec OpenSSH est très simple et permet de gagner en rapidité lors de l'administration de vos serveurs. Il est aussi très facile de donner des noms à vos connexions, grâce au fichier `config`.

