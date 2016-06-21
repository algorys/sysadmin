---
layout: post
title: Installer Puppet
lang: fr
ref: puppet
modified:
description: Comment installer puppet
tags: [tutoriel, puppet]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-03-01T09:52:05+01:00
---

# Introduction

Puppet, de Puppet Labs, est un outil qui aide les administrateurs à automatiser le provisionnement, la configuration et l'administration de leur infrastructure réseau. Utiliser ce genre d'outil permet de réduire le temps dépensé sur les tâches de base qui sont souvent répétitives et assure une consistance de votre infrastructure.

Puppet est disponible en deux versions : Puppet Enterprise et Puppet Open Source. Il fonctionne avec un système d'Agents, associés à un serveur Maître.

Ce tutoriel est basé sur celui de [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-puppet-to-manage-your-server-infrastructure). Je n'ai pas traduis, ni mis à jour la partie pour bloquer les versions de Puppet, mais si vous êtes en production, il vaut mieux effectuer cette opération.

# Prérequis

Pour suivre ce tutoriel, vous devez avoir un accès **root** sur les serveurs que vous allez manipuler. Vous devez également disposer d'un DNS ou y avoir accès. Si vous suivez ce tutoriel à des fins de tests, il est conseillé d'utiliser des machines virtuelles et de faire des snapshots de vos machines vierges. Ainsi, vous pourrez facilement les "remettre à zéro" lors de nouveaux essais.

Ce tutoriel couvre l'installation de Puppet sur des système Debian ou Ubuntu. Si vous avez une autre distribution, renseignez vous sur l'installation de Puppet pour celle-ci. Les fichiers de configurations de Puppet seront normalement sensiblement identiques.

Avant de poursuivre, il faut donc vous assurer que vous ayez :

* Un DNS privé : soit donné par le serveur de votre entreprise ou bien simuler sur vos serveurs grâce au fichier `hosts`.
* Les Ports Ouverts : le serveur Master de Puppet doit être accessible sur le port 8140.
* Un accès root sur les différents serveurs.

Je pars aussi du principe que vous avez les connaissances de bases pour administrer un ou plusieurs serveurs.

## Exemple d'infrastructure

Nous allons utiliser l'infrastructure suivante :

| Nom de l'hôte | Rôle   |  FQDN Privé              |
|:-------------:|:------:|:------------------------:|
| puppetone     | Client | puppetone.localdomain    |
| puppetsecond  | Client | puppetsecond.localdomain |
| puppetmaster  | Maître | puppetmaster.localdomain |

L'agent Puppet sera installé sur tous ces hôtes. Il seront tous référencés par leur nom de domaine.

Une fois que vous avez tous les prérequis, vous allez pouvoir monter le serveur Maître de Puppet.

# Créer le serveur Maître

Créer un nouveau serveur et ajoutez le à votre DNS.

| Nom de l'hôte | Rôle   |  FQDN Privé              |
|:-------------:|:------:|:------------------------:|
| puppetmaster  | Maître | puppetmaster.localdomain |

Utiliser "puppet" comme nom d'hôte pour le serveur maître simplifie grandement l'installation des agents par la suite, car c'est le nom par défaut que les agents essaieront de contacter.

Maintenant nous allons régler le service NTP du serveur.

## Installer NTP

Vu que Puppet utilise un système de certificat pour les noeuds (agents), le serveur Maître doit maintenir son temps à jour. Si les serveurs ont un temps qui divergent de trop, vous risquez d'avoir des problèmes de certificats qui ne sont plus à jour.

Mettez à jour vos dépôts et installez NTP :

```bash
sudo apt-get update && sudo apt-get -y install ntp
```

Il est plus pratique d'utiliser des `pools zones` qui sont proches de vous géographiquement. Ouvrez votre navigateur préféré, allez sur [NTP Pool Project](http://www.pool.ntp.org/en/) et choisissez une zone proche de vous. Nous utiliserons les serveurs Europe dans notre exemple.

Ouvrez le fichier de configuration de NTP (`sudo vi /etc/ntp.conf`) et ajoutez les serveurs suivants en tête de fichier :

```conf
server 0.europe.pool.ntp.org
server 1.europe.pool.ntp.org
server 2.europe.pool.ntp.org
server 3.europe.pool.ntp.org
```

Sauvegardez et quittez. Redémarrez le service NTP pour prendre en compte les modifications :

```bash
sudo service ntp restart
```

Maintenant que notre serveur a une heure à jour, nous pouvons installer Puppet Master.

# Installer Puppet Master

Il y a différentes façon d'installer la version Open Source de Puppet. Nous allons utiliser le paquet Debian nommé _puppetmaster-passenger_, qui est fourni par Puppet Labs. Le paquet _puppetmaster-passenger_ possède l'avantage d'inclure Puppet Master et un serveur Web de production déjà pré-configuré. Cela élimine pas mal d'étapes à l'installation par rapport au paquet _puppetmaster_ de base.

Télécharger le paquet fourni par Puppet Labs :

```bash
cd ~; wget https://apt.puppetlabs.com/puppetlabs-release-trusty.deb
```

Installez-le et mettez à jour la liste de vos paquets :

```bash
sudo dpkg -i puppetlabs-release-trusty.deb
sudo apt-get update
```

Enfin installez le paquet puppetmaster-passenger :

```bash
sudo apt-get install puppetmaster-passenger
```

Puppet Master, Passenger, Apache et les autres paquets nécessaires sont maintenant installés. Vu qu'on utilise Passenger avec Apache, le processus Puppet Master est contrôlé par Apache, dans cet exemple il tourne quand Apache est en route.

# Configurer les Noms et les Certificats

Puppet utilise des certificats SSL pour authentifier les communications entre le maître et les agents (noeuds). Le serveur Maître agit comme authorité de certification (CA) et doit généré son propre certificat qui signera les requêtes de certificats des agents.

## Effacez les Certificats existants

Effacez les certificats SSL existants qui ont été créés lors de l'installation du paquet. L'emplacement par défaut des certificats SSL est `/var/lib/puppet/ssl` :

```bash
sudo rm -rf /var/lib/puppet/ssl
```

## Configurer le Certificat

Lorsque vous créez le certificat, vous pouvez inclure tous les nom DNS vers lesquels vos agents pourront contacter le serveur Maître. Dans notre exemple, nous utiliseront donc respectivement "puppetmaster" et "puppetmaster.localdomain".

Ouvrez le fichier puppet.conf du serveur Maître (`sudo vi /etc/puppet/puppet.conf`). Cela devrait ressembler à ceci :

```conf
[main]
logdir=/var/log/puppet
vardir=/var/lib/puppet
ssldir=/var/lib/puppet/ssl
rundir=/var/run/puppet
factpath=$vardir/lib/facter
templatedir=$confdir/templates

[master]
# These are needed when the puppetmaster is run by passenger
# and can safely be removed if webrick is used.
ssl_client_header = SSL_CLIENT_S_DN
ssl_client_verify_header = SSL_CLIENT_VERIFY
```

Effacez la ligne `templatedir` car cette option est obsolète.

Ajoutez ensuite les deux lignes suivantes à la fin de la section `[main]` :

```conf
certname = puppet
dns_alt_names = puppetmaster,puppetmaster.localdomain
```

Il est important d'utiliser puppet pour `certname` car la configuration d'Apache attends un certificat qui s'appelle "puppet". Si vous décidez de changer de nom de certification, pensez à éditer le fichier de configuration correspondant (`/etc/apache2/sites-available/puppetmaster`) pour changer le nom du chemin du certificat.

Sauvegardez et quittez.

## Générer le nouveau Certificat

Maintenant, vous pouvez créer le nouveau certificat en tapant cette commande :

```bash
sudo puppet master --verbose --no-daemonize
```

Vous devriez voir plusieurs lignes indiquant que les clés SSL et les certificats sont créés. une fois que vous voyez la ligne `Notice: Starting Puppet master version X.X.X`, la création est finie. Faites `CTRL-C` pour quitter.

**Sample Output :**

```bash
Info: Creating a new SSL key for ca
Info: Creating a new SSL certificate request for ca
Info: Certificate Request fingerprint (SHA256): EC:7D:ED:15:DE:E3:F1:49:1A:1B:9C:D8:04:F5:46:EF:B4:33:91:91:B6:5D:19:AC:21:D6:40:46:4A:50:5A:29
Notice: Signed certificate request for ca
...
Notice: Signed certificate request for puppet
Notice: Removing file Puppet::SSL::CertificateRequest puppet at '/var/lib/puppet/ssl/ca/requests/puppet.pem'
Notice: Removing file Puppet::SSL::CertificateRequest puppet at '/var/lib/puppet/ssl/certificate_requests/puppet.pem'
Notice: Starting Puppet master version 3.6.2
```

Vous pouvez vous assurer de la bonne création du certificat en tapant :

```bash
sudo puppet cert --list --all
```

Cette commande vous sera utile pour afficher les certificats signés et ceux qui sont en attente de validation. Actuellement, seul le certificat du serveur Maître est affiché.

```bash
+ "puppet" (SHA256) 05:22:F7:65:64:CF:46:0E:09:2C:5D:FD:8C:AC:9B:31:17:2B:7B:05:93:D5:D1:01:52:72:E6:DF:84:A0:07:37 (alt names: "DNS:puppetmaster", "DNS:puppetmaster.localdomain")
```

Notre serveur Maître est prêt à fonctionner. Mais auparavant, regardons la configuration de plus près.

# Configurer Puppet Master

Le fichier de configuration principal de Puppet regroupe 3 sections : `[main]`, `[master]` et `[agent]`. Comme vous l'avez deviné, la section "main" contient la configuration globale, la section "master"est spécifique au serveur Maître et la section "agent" concerne les agents.

Ce fichier de configuration a de nombreuses options qui vont varier en fonction de votre configuration. Une description de ce fichier est disponible sur Puppet Labs : [Fichier de Configuration principal](https://docs.puppetlabs.com/puppet/latest/reference/config_file_main.html).

## Fichier Manifest Principal

Puppet utilise un language spécifique pour décrire ses configurations et celles-ci sont écrites dans un fichier appelé "manifeste", avec une extension `.pp`. Le manifeste par défaut se trouve dans `/etc/puppet/manifests/site.pp`. Nous verrons les bases de ce fichier plus tard, mais nous pouvons le créer de suite (si il n'existe pas déjà) :

```bash
sudo touch /etc/puppet/manifests/site.pp
```

## Démarrer Puppet Master

Nous pouvons maintenant mettre en route le serveur `puppetmaster` :

```bash
sudo service apache2 start
```

Votre serveur tourne mais pour le moment il ne s'occupe d'aucun agent. C'est ce que nous ferons dans la prochaine étape.

# Installer l'Agent Puppet

L'agent Puppet doit être installé sur tous les serveurs que le Maître doit gérer. Dans la plupart des cas, cela se résume souvent à l'installer sur tous vos serveurs. L'agent Puppet peut tourner sur la plupart des distributions Linux, quelques plateformes UNIX et Windows. Nous ne couvrirons que la partie Linux dans ce tutoriel et principalement sur les serveurs Ubuntu et Debian.

Les instructions pour installer Puppet sur les autres plateformes sont évidemment disponibles sur la doc de [Puppet Labs](http://docs.puppetlabs.com/puppet/3.8/reference/pre_install.html#next-install-puppet).

> **Note :** dans la suite de ce tutoriel, je pars du principe que vous avez déjà configuré tous les DNS des noeuds (serveurs) Puppet et qu'ils peuvent joindre le serveur Maître.

## Ubuntu / Debian

> **Note :** les deux serveurs puppetone et puppetsecond (ainsi que d'autres éventuels que vous aurez créés) suivront le même processus d'installation. Il faudra donc le répéter sur chacun d'entre eux.

Sur votre noeud Puppet, téléchargez le paquet Puppet Labs et installez-le :

```bash
cd ~; wget https://apt.puppetlabs.com/puppetlabs-release-trusty.deb
sudo dpkg -i puppetlabs-release-trusty.deb
sudo apt-get update
sudo apt-get install puppet
```

L'agent Puppet est désactivé par défaut, activez-le (`sudo vi /etc/default/puppet`) en changeant la clé `START` à "yes" :

```conf
START=yes
```

Sauvegardez le fichier et quittez.

## Configurer l'agent

Avant de lancer l'agent, il faut faire quelques modifications dans sa configuration (`sudo vi /etc/puppet/puppet.conf`) :

Le fichier devrait ressembler au fichier de configuration du serveur Maître. Effacez la ligne `templatedir` et la section `[master]`, ainsi que tout ce qui suit celle-ci.

En partant du principe que le serveur maître est joignable sur "puppet", l'agent devra pouvoir se connecter au maître tout seul. Si ce n'est pas le cas, il vous faudra ajouter le FQDN du serveur Maître. Je vous recommande d'ailleurs de procéder ainsi (remplacez le FQDN suivant par le votre) :

```conf
[agent]
server = puppetmaster.localdomain
```

Sauvegardez et quittez.

L'agent est prêt à être lancé :

```bash
sudo service puppet start
```

Si tout est configuré correctement, vous ne devriez voir aucune sortie dans votre console. La première fois que vous lancez l'agent Puppet, il génèrera un certificat et enverra un signal au serveur Maître. Une fois qu'il sera signé, le maître pourra communiquer avec l'agent.

> **Note :** si c'est votre premier agent, prenez le temps de signer le certificat avec le serveur Maître afin de vérifier que tout fonctionne bien. une fois cela fait, l'installation et les certificats des autres agents devraient se dérouler sans problème.

# Signer la Requête sur le Serveur Maître

La première fois que vous lancez un agent, il fera une demande de signature au près du serveur Maître. Avant de pouvoir communiquer avec ce noeud, il devra signer spécifiquement ce certificat.

## Lister les Requêtes de Certificats

Sur le serveur Maître, lister les requêtes non signées :

```bash
sudo puppet cert --list
```

Si vous avez juste fait un seul agent, vous devriez voir une sortie similaire à celle ci-dessous :

```bash
"puppetone.localdomain" (SHA256) B1:96:ED:1F:F7:1E:40:53:C1:D4:1B:3C:75:F4:7C:0B:A9:4C:1B:5D:95:2B:79:C0:08:DD:2B:F4:4A:36:EE:E3
```

Remarquez qu'il n'y a pas de petit `+` devant cette requête. C'est normal, cela indique qu'il n'est pas signé.

## Signer la Requête

Pour signer une requête, utiliser la commande `puppet cert sign` avec le nom d'hôte derriere. Dans notre exemple, ce sera `puppetone.localdomain` :

```bash
sudo puppet cert --sign puppetone.localdomain
```

Vous devriez avoir la sortie suivante si cela a réussi :

```bash
Notice: Signed certificate request for puppetone.localdomain
Notice: Removing file Puppet::SSL::CertificateRequest puppetone.localdomain at '/var/lib/puppet/ssl/ca/requests/puppetone.localdomain'
```

Le serveur Maître peut maintenant communiquer et contrôler le noeud qui correspond ce certificat.

Si vous voulez signer toutes les requêtes courantes, utilisez l'option `--all` :

```bash
sudo puppet cert --sign --all
```

## Révoquer les Certificats

Vous pouvez avoir envie de supprimer un hôte de Puppet ou en reconstruire un et le rappatrier dans Puppet. Dans ce cas, vous devrez révoquer le certificat de l'hôte sur le serveur Maître. Pour faire cela, utiliser l'option `--clean` :

```bash
sudo puppet cert --clean <hostname>
```

## Voir toutes les Requêtes Signées

Si vous voulez voir toutes les requêtes signées et non signées, exécutez la commande suivante :

```bash
sudo puppet cert --list --all
```

Vous devriez voir toutes les requêtes. Les requêtes qui ont été signées sont précédées d'un `+`, celles qui ne le sont pas n'ont rien devant :

```bash
  "puppetsecond.localdomain" (SHA256) E4:F5:26:EB:B1:99:1F:9D:6C:B5:4B:BF:86:14:40:23:E0:50:3F:C1:45:D0:B5:F0:68:6E:B2:0F:41:C7:BA:76
+ "puppetone.localdomain"    (SHA256) 71:A2:D3:82:15:0D:80:20:D4:7E:E3:42:C2:35:87:83:79:2B:57:1D:D5:5A:EC:F6:8B:EE:51:69:53:EB:6B:A1
+ "puppet"                   (SHA256) 05:22:F7:65:64:CF:46:0E:09:2C:5D:FD:8C:AC:9B:31:17:2B:7B:05:93:D5:D1:01:52:72:E6:DF:84:A0:07:37 (alt names: "DNS:puppetmaster", "DNS:puppetmaster.localdomain")
```

Félicitations ! Votre infrastructure est maintenant prête pour être gérée par Puppet !

# Démarrer avec Puppet

Maintenant que votre infrastructure est prête, nous allons voir quelques tâches de base qui vous aideront à démarrer.

## Comment les données sont recueillies ?

Puppet collecte les données sur ses différents noeuds grâce à un outil appellé _facter_. "_Facter_", par défaut, collecte des informations qui sont utiles à la configuration du système (par exemple : le nom de l'OS, le nom d'hôte, les adresses IP, les clés SSH, etc.). Il est possible d'ajouter des données supplémentaires pour optimiser vos configurations.

La collecte de ces données peut être utiles dans pas mal de situations. Par exemple, vous pouvez créer une configuration de serveur Web et définir l'adresse IP appropriée pour un hôte virtuel particulier. Ou vous pouvez préciser que votre serveur est un Ubuntu Server, donc vous pouvez démarrer le service `apache2` plutôt que `httpd`. Cela vous donne déjà une idée de ce que vous pouvez faire avec ces données.

Pour voir la liste des données qui sont automatiquement collectées sur votre noeud (agent), vous pouvez utiliser cette commande :

```bash
facter
```

## Comment le Manifeste Principal est exécuté ?

L'agent Puppet vérifie périodiquement le serveur Maître (en général toutes les 30 minutes). Durant ce temps, il envoie les données qui le concerne au maître, et "tire" le catalogue courant (une liste compilée des ressources désirées et leurs états voulus, déterminée par le manifeste principal). L'agent "noeud" essaie ensuite d'appliquer les changements pour arriver à l'état désiré. Ce cycle se répète tant que le Maître est en ligne et communique avec les agents "noeuds".

## Exécution Immédiate sur un Agent

Il est aussi (heureusement) possible d'initialiser cette vérification sur un agent particulier grâce à cette commande (sur le noeud en question) :

```bash
puppet agent --test
```

L'éxécution de cette commande applique le Manifeste principal sur l'agent. Vous devriez voir une sortie similaire à celle-ci :

```bash
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppetone.localdomain
Info: Applying configuration version '1407966707'
```

## Manifestes isolés

La commande `puppet apply` permet d'exécuter des manifestes qui ne sont pas ratachés au manifeste principal. Cela applique uniquement le manifeste au noeud sur lequel vous exécuter cette commande. Voici un exemple :

```bash
sudo puppet apply /etc/puppet/modules/test/init.pp
```

Appliquer les manifestes de cette façon est utile si vous voulez seulement le tester sur un noeud ou bien si vous voulez le faire qu'une seule fois (Et non périodiquement comme vu ci-dessus).

# Un Manifeste Simple

Comme dit précédemment dans cet article, le Manifeste principal se trouve dans `/etc/puppet/manifests/site.pp`.

> **Astuce :** si vous utilisez vim pour éditer vos fichiers, je vous conseille d'installer le paquet `vim-puppet`. Cela vous permettra d'avoir la coloration syntaxique des fichiers `.pp`. Pour cela installer le paquet `sudo apt-get install -y vim-puppet` et appliquez-le à vim `vim-addons install puppet`.

Sur le Maître, ouvrez le Manifeste principal (`sudo vi /etc/puppet/manifests/site.pp`) et ajoutez les lignes suivantes pour décrire un fichier de ressource :

```puppet
file {'/tmp/example-ip':                                            # Type : fichier de ressource, et son nom
  ensure  => present,                                               # S'assure qu'il existe
  mode    => 0644,                                                  # Les permissions du fichier
  content => "Voici mon adresse IP publique : ${ipaddress_eth0}.\n",  # Note la donnée ipaddress_eth0 
}
```

Maintenant sauvegardez et quittez. Chaque ligne de commentaire explique la ressource que nous avons définie. En résumé, cela **assure** que tous les agents "noeud" ont un fichier dans `/tmp/example-ip`, avec les permissions `-rw-r--r--` et que le text contient l'IP du noeud.

Vous pouvez attendre que l'agent vérifie son catalogue ou bien appliquer la configuration avec `puppet agent --test` sur l'un d'entre eux.

Ensuite, affichez le fichier créé :

```bash
cat /tmp/example-ip
```

Vous devriez voir la sortie suivante :

```bash
Voici mon adresse IP publique : 128.131.192.11.
```

## Spécifiez un noeud

Si vous voulez définir une ressource pour un noeud spécifiquement, vous devez le faire sur le manifeste (`sudo vi /etc/puppet/manifests/site.pp`) et rajouter les lignes suivantes :

```puppet
node 'puppetone', 'puppetsecond' {    # Applique sur les noeuds puppetone et puppetsecond
  file {'/tmp/dns':    # Type de ressource et son nom
    ensure => present, # S'assure qu'il existe
    mode => 0644,
    content => "Seuls les serveurs DNS auront ce fichier.\n",
  }
}

node default {}       # S'applique à des noeuds qui ne sont pas explicitement définis
```

Sauvegardez et quittez.

Maintenant Puppet s'assure qu'un fichier `/tmp/dns` existe sur _puppetone_ et _puppetsecond_. Vous pouvez exécuter `puppet agent --test` sur ces deux agents si vous ne voulez pas attendre qu'ils "tirent" leur catalogue.

Ces exemples ne sont dans la réalité pas très utiles (en effet ils ne font rien), mais cela prouve que Puppet fonctionne bien.

# Utiliser un Module

A présent, nous allons utiliser un module. Les modules sont utiles pour regrouper plusieurs tâches. Il existe déjà de nombreux modules écrit par la communauté Puppet, et vous pouvez écrire le votre.

Sur le Maître, installez le module `puppetlabs-apache` de _forgeapi_.

```bash
sudo puppet module install puppetlabs-apache
```

**Attention :** n'utilisez pas le module Apache sur une installation d'Apache existante. Cela effaçera toutes les configurations qui ne sont pas gérées par Puppet !

Éditez le fichier `site.pp` (`sudo vi /etc/puppet/manifest/site.pp`) et rajoutez les lignes suivantes :

```puppet
node 'puppetsecond' {
  class { 'apache': }             # Utilise le module Apache
  apache::vhost { 'example.com':  # Définit l'hôte virtuel
    port    => '80',              # Renseigne le port
    docroot => '/var/www/html'    # Indique la racine du vHost
  }
}
```
Sauvegardez et quittez. maintenant la prochaine fois que Puppet mettra à jour puppetsecond, il installera le paquet `apache2`, configurera l'hôte virtuel appelé "example.com", qui écoutera sur le port 80 et aura un document "root" situé dans `/var/www/html`.

Si vous exécutez la commande `sudo puppet agent --test` sur puppetsecond, vous aurez une sortie qui affichera l'installation d'Apache. Une fois terminé, vous pouvez aller sur l'adresse IP de votre noeud dans votre navigateur. Cela affichera la page par défaut d'Apache.

Félicitations ! Vous avez utilisé votre premier module Puppet !

# Conclusion

Vous savez comment avoir une installation basique de Puppet. Je ferais bientôt un tutoriel plus avancé sur les manifestes et les modules.

