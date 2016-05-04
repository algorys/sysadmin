---
layout: post
title: Installer Prosody
lang: fr
ref: prosody
modified:
description: Comment installer Prosody
tags: [tutoriel, prosody]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2016-04-27T14:15:05+01:00
---

# Introduction

Ce tutoriel vous aidera à monter une messagerie instantanée avec [Prosody](https://prosody.im/) en interne. Prosody est un serveur de communication utilisant le protocole [XMPP](https://en.wikipedia.org/wiki/XMPP) qui permet de faire de la messagerie instantanée que ce soit pour vous ou pour votre entreprise, il est utilisable sur tout les types de terminaux. Le fait d'utiliser un protocole standard permet de l'utiliser avec n'importe quel client XMPP.

> **Info** : une liste des clients XMPP plus ou moins à jour est disponible sur [Wikipédia](https://fr.wikipedia.org/wiki/Liste_de_clients_XMPP). Personnellement, j'utilise [Gajim](https://gajim.org/index.fr.html) sur Linux et [Jitsi](https://jitsi.org/) sur Windows, mais à vous de voir quel client s'adapte le mieux à vos besoins.

# Prérequis

Vous aurez besoin seulement d'un serveur Linux avec un accès `root` sur celui-ci. J'expliquerai également comment lier les comptes à un Active Directory.

# Installation de Prosody

## Méthode 1

Pour installer Prosody, vous pouvez le faire via les paquets :

```bash
sudo apt-get update
sudo apt-get install prosody
```

Sur Ubuntu Server (14.04.4 LTS), le dépôt est bien à jour et propose la dernière version. 

## Méthode 2

Vous pouvez aussi ajouter le dépôt de Prosody ce qui sera plus facile pour se tenir à jour :

```bash
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add -
sudo apt-get update
sudo apt-get install prosody
```

# Configuration

Vous allez devoir maintenant configurer Prosody. La première chose à faire est de créer un _hôte virtuel_ dans un fichier de configuration. Les fichiers de configuration de Prosody portent l'extention `.lua`.

Allez dans le répertoire de Prosody et regarder ce qui se trouve dans le dossier :

```bash
cd /etc/prosody
ls -l
```

Vous devriez avoir les dossiers suivants :

* `certs` qui contiendra vos certificats et vos clés.
* `conf.d` qui aura les configurations actives (un peu comme Apache ou Nginx) grâce à des liens symboliques.
* `conf.avail` : les fichiers de configuration disponibles.

Vous allez maintenant créer un fichier de configuration.

Remplacez `mail.fr` par le nom de votre domaine afin de correspondre à votre configuration. Ou par l'email, afin que les utilisateurs se connectent avec leur email à la place de leur login Active Directory. De plus cela permet d'afficher les status des gens dans Outlook (voir plus loin). Je partirais donc sur cet exemple dans la suite de ce tutoriel.


```bash
# Si le dossier n'existe pas
sudo mkdir conf.avail
# Créez le fichier de configuration
sudo vi conf.avail/mail.fr.cfg.lua
```

Ensuite remplissez comme suit :

```lua
--- Nom de l'hôte virtuel ---
VirtualHost "mail.fr"

--- Components ---
Component "salons.mail.fr" "muc"
    name = "Serveur de Salons pour mail.fr"
    restrict_room_creation = "admin"
```

Votre hôte virtuel sera l'adresse à utiliser pour vous connecter : par exemple `user@mail.fr`. 

Vous pouvez aussi désactiver l'hôte en rajoutant la ligne `enabled = false`.

Les `Components` sont des modules que vous pouvez rajouter au serveur prosody. Dans ce cas, on défini une `MUC` ("Multi-user Chat" : en somme une "room"), avec une adresse correspondante (`salons.mail.fr`).

Le fait de mettre `admin` pour `restrict_room_creation` autorise seulement les administrateurs de Prosody à créer des salons **persistants** ! Cela veut dire que les autres utilisateurs ne pourront que créer des conférences temporaires. La plupart des clients XMPP proposent ensuite aux administrateurs de configurer les salons selon leurs envies. Si vous ne voulez pas de cette fonctionnalité, supprimer simplement la ligne.


# Authentification

Voyons maintenant comment définir un administrateur pour Prosody. Ouvrez le fichier de configuration principal (`sudo vi prosody.cfg.lua`) et ajoutez votre utilisateur. Vous pouvez aussi en profiter pour parcourir le fichier. Il est assez bien expliqué dans son ensemble et permet de s'y retrouver assez rapidement.

```lua
admins = { "admin@mail.fr" }
```


Prosody peut gérer l'authentification grâce à votre serveur AD. Comme dit plus haut, nous allons faire en sorte que les gens puissent se connecter avec leur email. Afin de réaliser cela, il faut que l'email des personnes soit bien évidemment renseigné dans votre AD.

## Cyrus et Saslauthd

Pour gérer l'authentification, nous aurons besoin de [Cyrus](https://cyrusimap.org/mediawiki/index.php/Cyrus_SASL). Voici comment l'installer :

```bash
sudo apt-get install lua-cyrussasl sasl2-bin
```

Puis nous allons l'indiquer dans notre configuration (`vi conf.avail/mail.fr.cfg.lua`) et votre fichier devra ressembler à cela :

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

Il faut ensuite éditer le fichier de configuration de **saslauthd** (`sudo vi /etc/default/saslauthd`) :

```conf
[...]
START=yes
[...]
MECHANISMS="ldap"
[...]
# Here we set the ldap option file
MECH_OPTIONS="/etc/saslauthd.conf"
```

Maintenant, il faut définir les options avec lesquelles saslauthd va se connecter à votre AD. Ajoutez et éditez le fichier `/etc/saslauthd.conf` :

```conf
ldap_servers: ldap://ip_server
ldap_search_base: dc=domain,dc=com

ldap_bind_dn: DOMAIN\user
ldap_bind_pw: password
ldap_start_tls: no
ldap_auth_method: bind

# Le filtre ci-dessous sert à des connexions avec le nom d'utilisateur
# ldap_filter: (samAccountName=%U)
# Ici nous utiliserons le mail :
ldap_filter: (mail=%U@mail.fr)
```

Remplacez évidemment les différentes options avec votre configuration, enregistrez et quittez.

### Tester l'authentification

Vous devez redémarrer le service saslauthd :

```bash
sudo service saslauthd restart
```

Maintenant, vous pouvez tester la configuration avec la commande suivante :

```bash
# Dans le cas ou l'utilisateur aurait un mail "u.user@mail.fr"
sudo testsaslauthd -u u.user -p password
```

Si vous avez la sortie suivante : `0: OK "Success."` votre authentification fonctionne. Dans le cas contraire, vérifier bien vos options pour saslauthd et  si vous avez bien installer toutes les dépendances.

## Prosody et Sasl

Pour que l’utilisateur prosody ait accès à la socket de saslauthd, on prendra soin de l’ajouter au groupe sasl :

```bash
sudo gpasswd -a prosody sasl
```

Vous pouvez vérifier que l'utilisateur prosody a accès à saslauthd :

```bash
sudo su prosody -s /bin/bash
/usr/sbin/testsaslauthd -u u.user -p password
```

Enfin, il nous faut ajouter un dernier fichier de configuration dans `/etc/sasl/`. Le dossier n'existe probablement pas, vous devrez donc le créer :

```bash
cd /etc/
sudo mkdir sasl
# Puis éditez le fichier qui suit
sudo vi /etc/sasl/prosody.conf
```

Ajoutez-y les deux lignes suivantes :

```conf
pwcheck_method: saslauthd
mech_list: PLAIN
```

Enregistrez et quittez.

Redémarrez maintenant saslauthd et prosody :

```bash
sudo service saslauthd restart
sudo service prosody restart
```

# Tester un client

Vous allez maintenant pouvoir tester avec un client (Gajim, Jitsi, Pigdin...). Il vous suffira de rentrer :

* l'IP du serveur XMPP, en laissant le port 5222.
* Vos identifiants : mail et mot de passe
* Accepter le certificat

Dans la plupart des clients vous pourrez configurer un Proxy si besoin.

Normalement, la connexion devrait s'établir et votre statut passer en **connecté** ! Si ce n'est pas le cas, regardez vos logs :

```bash
sudo tail -f /var/log/prosody/prosody.log
# ou bien
sudo tail -f /var/log/prosody/prosody.err
```

Les dossiers des logs peuvent être définis dans le fichier `prosody.cfg.lua`.

# Gérer les Certificats

Avec Prosody, vous pouvez définir des certificats (auto-signé ou non) pour chaque hôte virtuel (`VirtualHost`). Dans le fichier de configuration principal (`prosody.cfg.lua`), vous devriez avoir les lignes suivantes :

```lua
-- These are the SSL/TLS-related settings. If you don't want
-- to use SSL/TLS, you may comment or remove this
ssl = {
    key = "/etc/prosody/certs/localhost.key";
    certificate = "/etc/prosody/certs/localhost.crt";
}
```

Ces lignes indiquent le chemin vers la clé et le certificat pour le serveur **localhost**, ainsi que pour votre serveur Prosody. Il est donc défini 2 fois. Comme vous pouvez le remarquer, le serveur **localhost** est désactivé (`enabled = false`). Cela ne gène donc pas.

Par contre, pour votre nouveau serveur virtuel (`mail.fr`), il serait bien de le redéfinir.

Le but ici est d'avoir un certificat faisant autorité (`localhost`) et qui signera donc nos autre certificats. 

## Installer Lua-expat 1.3

Si vous avez un problème de version pour lua-expat, il vous faut rajouter les dépôt universe de **vivid** (`sudo vi /etc/apt/sources.list`) :

```conf
deb http://us.archive.ubuntu.com/ubuntu vivid main universe
```

Puis on met à jour les dépôts et on installe lua-expat :

```bash
sudo apt-get update
sudo apt-get install lua-expat
```

> **Note :** durant la mise à jour des paquets, il est possible que vous ayez des erreurs. Installez **lua-expat** et vous pourrez ensuite commentez la ligne rajoutée dans `sources.list`.

## Générer un fichier de requête

Maintenant afin de pouvoir générer notre certificat ssl pour notre hôte virtuel **mail.fr**, il faut d'abord un fichier de requête :

```bash
sudo prosodyctl cert request mail.fr
```

Tapez `Entrée` pour valider les données par défaut ou rentrez de nouvelles informations. Cela devrait ressembler à quelque chose comme ça :

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

## Signer avec le CA

Le certificat d'autorité va mainitenant nous servir à signer et créer notre clé :

```bash
sudo openssl x509 -req -days 730 -in /var/lib/prosody/mail.fr.req -CA /etc/prosody/certs/localhost.crt -CAkey /etc/prosody/certs/localhost.key -set_serial 01 -out /var/lib/prosody/mail.fr.crt
```

Vous devriez avoir la sortie suivante :

```bash
Signature ok
subject=/C=FR/L=The Internet/O=MyBusiness/OU=DSI/CN=mail.fr/emailAddress=m.mail@mail.fr
Getting CA Private Key
```

## Ajouter le certificat

Maintenant que nous avons un beau certificat auto-signé, nous pouvons le rajouter à notre hôte virtuel. Ouvrez le fichier de configuration (`sudo vi conf.avail/mail.fr.cfg.lua`) et rajoutez-le comme suit :

```lua
ssl = {
        key = "/var/lib/prosody/mail.fr.key";
        certificate = "/var/lib/prosody/mail.fr.crt";
}
```

> **Note :** faites bien attention rajouter ces lignes après la déclaration de `VirtualHost`. Si vous les rajouter avant, cela ne marchera pas.

Maintenant, vous pouvez redémarrer Prosody pour appliquer les changements :

```bash
sudo service prosody restart
```

Lors de la reconnection de votre client, vous devriez avoir une nouvelle demande d'acceptation du certificat. Vous pouvez lors de celle-ci l'afficher afin de voir les données que vous avez renseignez plus haut.

Validez et acceptez. Cochez la case **ignorer** si vous ne souhaitez plus avoir cet avertissement.

Vous pourrez par la suite répétez ces manipulations si vous avez d'autres hôtes virtuels à rajouter. Signez bien vos certificats avec la même autorité (localhost dans ce cas) !

# Autres possibilités

Prosody permet de faire beaucoup de choses, comme créer des salons persistants, afficher des messages de bienvenue, recevoir des fichiers etc...

## Les salons

Pour la création de salons, vous avez normalement écrit les lignes suivantes au début du tutoriel :

```conf
--- Components ---
Component "salons.mail.fr" "muc"
    name = "Serveur de Salons pour mail.fr"
    restrict_room_creation = "admin"
```

Cela veut dire que la création de salons ne sera possible que par les administrateurs. D'autre part, les salons sont créés par le client !

Vous aurez la possibilité dans votre client de définir si le salon est permanent, ouvert, protégé par un mot de passe, etc... Les autres personnes connectées pourront ensuite rechercher les salons disponibles dans leur client et s'y connecter si ils ont les droits suffisants.

## Le mot du jour

Vous pouvez définir un mot du jour pour votre messagerie, en rajoutant la ligne suivante (dans `prosody.cfg.lua`) :

```lua
motd_text = [[Bienvenue sur la Messagerie Instantanée.]]
```

Redémarrez ensuite **prosody** pour appliquer le changement.

## Chemins des Plugins

Si l'envie vous prends de rajouter d'autre plugins, vous pouvez définir le chemin de ceux-ci grâce à la ligne suivante :

```lua
plugin_paths = { "/usr/lib/prosody/modules", "/usr/lib/prosody/prosody-modules"}
```

Redémarrez ensuite **prosody** pour appliquer le changement.

## Chemins des configurations

Vous pouvez également préciser les chemins vers vos configurations (toujours dans le fichier de configuration principal) :

```lua
Include "conf.d/*.cfg.lua"
```

# Conclusion

Au final, Prosody est très complet. Il existe évidemment d'autre serveur XMPP mais je trouve que Prosody propose tout ce qu'il faut à une messagerie instantanée et permet de configurer vos connexions avec précision.
