---
layout: post
title: FusionInventory pour GLPI
lang: fr
ref: fusioninventory
modified:
description: tutoriel FusionInventory pour GLPI
tags: [tutoriel, fusioninventory, glpi]
image:
  feature:
  credit:
  creditlink:
author: algorys ddurieux
comments: true
share:
date: 2016-03-14T13:20:05+01:00
---

# Introduction

[FusionInventory pour GLPI](https://github.com/fusioninventory/fusioninventory-for-glpi) est un plugin GLPI permettant de collecter des données à partir d'agents [FusionInventory](http://fusioninventory.org/) installés sur des hôtes. 
Il permet de mettre à jour les données se trouvant dans l'inventaire de GLPI et cela de manière automatique. 
Les agents permettent :

* l'inventaire hardware et software des ordinateurs, serveurs, machines virtuelles
* l'inventaire hardware et software des smartphones sous Android
* l'inventaire des équipements réseau via SNMP (switch, routeur, imprimante réseau...)
* la découverte réseau (découvrir tout ce qui est connecté au réseau)
* le déploiement d'application sur les ordinateurs
* le wake on lan (réveil des ordinateurs à distance)
* l'inventaire des serveurs ESX (sur chaque hote ESX ou via le vcenter)


Les agents peuvent être déployés par stratégie (GPO) ou via des scripts et fonctionnent sur la plupart des OS (Windows, Mac OS X, Linux, *BSD...).

Ce genre de processus peut faciliter grandement l'inventaire de votre parc informatique et donc la vie des "Sysadmin".

# Prérequis

Pour suivre ce tutoriel, il vaut mieux avoir un serveur [GLPI](/tuto/glpi-installation/) et de préférence en version `0.90.1` (donc à jour). 
Cependant les autres versions de GLPI devraient avoir une configuration quasi similaire.

* Un accès `root` / `administrateur` sur ce serveur sera nécessaire.
* Il vous faut aussi un (ou des) serveur/client Linux sur lequel installer l'agent.
* Un (ou des) serveur/client Windows pour installer un agent

> Il n'est pas forcément obligatoire d'avoir un Windows et un Linux pour suivre ce tutoriel, mais les deux peuvent servir.

# Récupération de l'archive

## Méthode simple

Prendre la dernière version stable sur la [page des releases](https://github.com/fusioninventory/fusioninventory-for-glpi/releases)

```bash
wget https://github.com/fusioninventory/fusioninventory-for-glpi/releases/download/glpi090%2B1.2/fusioninventory-for-glpi_0.90.1.2.tar.gz
tar zxvf fusioninventory-for-glpi_0.90.1.2.tar.gz
sudo cp -R fusioninventory/ /var/www/glpi/plugins/
```

> Ne surtout pas donner les droits d'écriture à apache sur ce dossier (sécurité ^^)

Voilà votre dossier est prêt !


## Méthode développeur avec git

Pour récupérer le plugin FusionInventory pour GLPI, vous pouvez passez par Git. Installez-le si ce n'est pas déjà fait :
```bash
sudo apt-get install -y git
```

Puis récupérer le dépôt et basculer sur le `tag` voulu (c'est à dire sur la dernière release) :

```bash
sudo cd /var/www/glpi/plugins/ && git clone https://github.com/fusioninventory/fusioninventory-for-glpi.git fusioninventory
sudo cd /var/www/glpi/plugins/fusioninventory && git checkout glpi090+1.2
```

> Ne surtout pas donner les droits d'écriture à apache sur ce dossier (sécurité ^^)

Voilà votre dossier est prêt !

# Activer le plugin

Allez sur l'interface Web de GLPI, connectez-vous et rendez-vous dans **Configuration => Plugins**. 
Cliquez sur **Installer** puis sur **Activer**. Votre plugin devrait maintenant être opérationnel.

Il faut maintenant le configurer. 
C'est un peu caché, il suffit d'aller dans **Administration => Entités => Root entity** et de cliquer sur l'onglet `Fusioninventory`. 

> `Root entity` est normalement l'entité principale par défaut dans GLPI. Si votre configuration diffère, appliquez les changements en conséquence.

Dans ce panneau, remplissez le champ `URL d'accès au service` avec votre url (ou votre IP. Vérifiez bien que l'url est correcte et accessible !). 
L'url à mettre est celle de GLPI, donc elle doit s'arrêter au glpi (exemple : http://ipserveur/glpi).
Puis sauvegarder !

Félicitation, votre plugin est prêt à repérer ses agents.

# Installer un agent

## Windows

L'installation sur Windows est assez simple, il suffit de télécharger le client sur la [page dédiée](http://forge.fusioninventory.org/projects/fusioninventory-agent-windows-installer/files) et de choisir le bon `.exe`.

Ensuite vous n'avez qu'à suivre l'installation et indiquer l'url de votre serveur GLPI correspondante :

`http://ipserveur/glpi/plugins/fusioninventory/`

ou si vous avez un DNS :

`http://fqdn/plugins/fusioninventory/`

## Linux

Pour Linux, c'est un peu différent, il va falloir rajouter les dépôts et installer à partir de ceux-ci ou faire une installation à partir des sources.

### Installation par les dépôts

Lors de cette manipulation, vous vous assurez d'avoir une version stable qui sera facile à mettre à jour. Récupérez la clé GnuPG avec la commande suivante :

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 049ED9B94765572E
```

> **Note :** La documentation officielle indique l'adresse `keyserver.ubuntu.com` sans port ni URI. 
Toutefois sous Ubuntu Server j'ai du forcer le port 80 et rajouter `hkp://` pour que la commande aboutisse.

Si vous n'avez pas l'outil `lsb-release`, installez-le :

```bash
sudo apt-get install lsb-release
```

Une fois cela fait, on peut rajouter le dépôt dans le fichier `sources.list` :

```bash
echo "deb http://debian.fusioninventory.org/debian/ `lsb_release -cs` main" >> /etc/apt/sources.list
```

Mettez à jour vos paquets et installez l'agent

```
sudo apt-get update
sudo apt-get install fusioninventory-agent
```

### Installation à partir des sources

Pour l'installer à partir des sources, il faut déjà s'occuper des dépendances requises à installer :

```
sudo apt-get install dmidecode nmap make
```

Et ensuite, il va falloir "quelques" dépendances `perl` :

```bash
sudo apt-get install libmodule-install-perl libmodule-build-perl libhttp-server-simple-psgi-perl libhttp-proxy-perl libio-captureoutput-perl libipc-run-perl libnet-snmp-perl libnet-telnet-cisco-perl libpoe-component-client-dns-perl libpoe-component-resolver-perl libtest-compile-perl libtest-deep-perl libtest-exception-perl libtest-most-perl libhttp-server-simple-authen-perl libio-capture-perl libio-captureoutput-perl libpoe-component-client-ping-perl libtest-http-server-simple-perl libtest-mockmodule-perl libtest-mockobject-perl libtest-nowarnings-perl libtest-failwarnings-perl libtest-warnings-perl libfile-copy-recursive-perl libxml-treepp-perl libproc-daemon-perl libproc-pid-file-perl
```

Récupérez maitenant l'archive qui vous intéresse, ici la 2.3.17 :

```bash
wget https://github.com/fusioninventory/fusioninventory-agent/archive/2.3.17.tar.gz
tar xzvf 2.3.17.tar.gz
cd fusioninventory-agent-2.3.17
```

Puis on installe tout ça :

```bash
perl Makefile.PL
make
sudo make install
```

Voilà, vous devriez avoir après la commande Perl un résumé des dossiers utilisés :

```bash
Installation summary
--------------------
prefix: /usr/local
configuration installation directory: /usr/local/etc/fusioninventory
constant data installation directory: /usr/local/share/fusioninventory
variable data installation directory: /usr/local/var/fusioninventory
```

Et à la fin de `make install`, vous devriez avoir une information très importante : où se trouve le fichier de configuration de l'agent !

```bash
if [ -f //usr/local/etc/fusioninventory/agent.cfg ]; then \
        install -m 644 etc/agent.cfg /usr/local/etc/fusioninventory/agent.cfg.new; \
    else \
        install -m 644 etc/agent.cfg /usr/local/etc/fusioninventory/agent.cfg; \
    fi
```

### Dépendances Perl

Il est possible qu'il vous manque encore quelques dépendances lorsque vous lancerez l'agent pour la première fois. 
Pour résoudre ce genre de problème, il suffit d'installer les modules manquants avec la commande suivante en remplaçant `MODULE:MODULE` par le nom correspondant :

```bash
sudo perl -MCPAN  -e 'install "MODULE:MODULE"'
```

Actuellement, le `core` demande au minimum les dépendances suivantes :

* Config::Tiny
* File::Which
* LWP::UserAgent
* Net::IP
* Text::Template
* UNIVERSAL::require
* XML::TreePP

Vous pouvez vérifier sur le [dépôt de l'agent](https://github.com/fusioninventory/fusioninventory-agent) pour voir toutes les dépendances requises. 

### Configurer l'agent

En fonction de votre installation, vous devriez avoir un fichier de configuration pour votre agent dans un dossier `FOLDER/fusioninventory/agent.cfg`. 
Ouvrez-le et décommentez la ligne `#server`. Modifiez ensuite l'url pour correspondre à votre installation GLPI :

```conf
[...]
# send tasks results to a FusionInventory for GLPI server
server = http://glpi_fqdn/plugins/fusioninventory/
[...]
```

## Lancer l'agent

Maintenant que vous avez votre agent installé, vous pouvez le lancer. Pour cela il suffit de taper la commande suivante :

```bash
sudo fusioninventory-agent
```

Vous devriez avoir la sortie suivante qui vous indique ce que fait l'agent :

```bash
[info] sending prolog request to server server0
[info] running task Deploy
[info] running task ESX
[info] ESX support disabled server side.
[info] running task Inventory
[info] running task Collect
```

Vous pouvez également lancer une découverte du réseau, en indiquant une IP de début et une IP de fin :

```bash
# Vous pouvez ajouter --debug si besoin
sudo fusioninventory-netdiscovery --first x.x.x.x --last x.x.x.x
```

Cela devrait vous afficher les matériels trouvés.

# Programmer une tâche

Il va falloir importer tout ça dans GLPI et pour ça, il faut configurer une tâche dans celui-ci. 
Mais avant de commencer, faites une vérification de la bonne détection de votre agent sur GLPI. 
Rendez-vous sur l'interface web de GLPI et allez dans **Plugins => FusionInventory**. 
Allez dans **Général => Gestion des agents**. 
Si vous avez déjà lancé votre agent une fois (voir ci-dessus), vous devriez voir apparaître votre agent sur cette page. 
Si ce n'est pas le cas, vous avez dû faire une erreur quelque part, vérifiez les étapes précédentes.

## Créer une tâche

Maintenant, aller dans **Tâches => Gestion des tâches**. 
Vous devriez avoir une page vide car vous n'en avez pas encore créée. 
Cliquez sur le gros **+** à côté de **Gestion des tâches** et donnez un nom à votre tâche (par exemple "découverte") puis cliquez sur **Ajouter**.

Celà a du vous créer votre tâche. N'oubliez pas de cocher la case **Actif**.

Allez dans l'onglet **Configuration des jobs**. Cliquez sur **Ajoutez un job** !

## Ajouter un job

Pour ajouter un job, il va falloir définir pas mal de choses :

* Donnez-lui un nom (par exemple "découverte LAN") et sélectionnez une méthode. Ici prenez "Découverte réseau", puis faites **Ajouter**.
* Cliquez sur le nom de votre job fraichement créé. Vous pouvez voir que maintenant vous pouvez ajouter des cibles et des acteurs.
  * Les "cibles" sont ce que vous allez découvrir
  * Les "acteurs" sont en les agents qui vont effectuer le job.
* Rajouter une cible en cliquant sur le **+**, sélectionnez **Plage IP**. Puis choisissez une plage que vous avez définie. 
Si vous n'aviez pas encore de plage IP de défini, cliquez sur le **i** en dessous. 
Cela vous ouvrira un nouvel onglet vous permettant de définir une plage IP. 
Bien entendu, il faut que vous définissiez une plage que l'agent peut voir et qui existe ! Puis faites **Ajouter cible**.
* Pour les acteurs, il suffit de rajouter un agent que vous avez installé et ensuite de cliquer sur **Ajouter Acteur**.
* Une fois les deux étapes précédentes effectuées, cliquez sur **Mettre à jour** ! Ne quittez pas la page avant d'avoir fait cela, sinon votre configuration ne sera pas sauvegardée !

Si vous retournez dans l'accueil de la Gestion des tâches, vous devriez voir la tâche que vous avez créée.

## Démarrer la tâche

Si vous allez dans **Tâches => Monitoring / logs**, vous devriez voir votre tâche avec plusieurs noms d'étapes en dessous et des zéros en face de chacun des noms. 
Vous pouvez voir que pour le moment la tâche est inactive. Le but maintenant va être de mettre votre tâche en _Préparé_ !

Pour ça on va changer d'endroit, allez dans **Configuration => Actions automatiques**. 
Cela va vous afficher différentes actions que GLPI va exécuter en fonction de ses différents plugins / tâches. 
Cherchez l'action qui s'appelle **taskscheduler** et cliquez dessus !

> Dans d'autres versions de GLPI et de FusionInventory, il me semble que l'action s'appelle différemment. 
Regardez bien si elle est liée à Fusioninventory, normalement c'est indiqué.

Ça n'a l'air de rien comme ça, mais beaucoup d'_entre nous_ ont galéré pour trouver comment faire cette manipulation. 
Une fois sur la page de l'action **taskscheduler**, cliquez sur **Éxécuter** !

Si vous retournez sur votre tâche de Fusioninventory, onglet **Monitoring / Logs**, elle devrait être passée en **Préparé** ! 
Si ce n'est pas le cas, vous pouvez sélectionner juste au-dessus l'**Intervalle de rafrachissement** des tâches. En général, 5 minutes suffisent.

Maintenant, retournez sur votre console et lancez l'agent :

```
sudo fusioninventory-agent
```

Et suivez le déroulement de votre tâche dans GLPI. Elle devrait passer **En cours** (en violet notamment). 
Vous pouvez cliquez dessus, cela affichera votre agent et le job qu'il effectue. 
Vous pouvez ensuite cliquer sur le nom de votre agent pour voir plus de détails.

En fonction de la plage que vous avez définie, la vitesse de votre réseau et le nombre de matériels à trouver, la tâche peut prendre plus ou moins de temps avant de passer en **Succès** (vert).

## Voir le matériel trouvé


```
YYYY-DD-MM HH:MM:SS Ok  Ts que l'agent a terminé, vos matériels sont ajoutés automatiquement à GLPI.
FusionInventory devrait afficher (dans les logs) le message :

```bash
YYYY-DD-MM HH:MM:SS Ok  Total Found:XX Created:XX Updated:X
```

Si ce n'est pas le cas ou que le plugin vous indique le message `Import Denied`, cela veut dire que les **Règles d'import de matériel** du plugin sont trop restrictive pour infrastructure. 
En effet, FusionInventory a des règles par défaut définies dans **Règles => Règles d'import et de liaison des matériels**. 
Votre inventaire peut avoir réussi, mais FusionInventory ne les a pas "validé" et les a mis dans une sorte de blacklist dans **Règles => Matériels ignorés à l'import**.

Si le plugin vous a bien importé votre matériel, vous n'allez pas le retrouver directement dans **Parc**. 
Il faut que vous alliez dans **Parc => Equipement non-géré** et là surprise, vous avez tous vos équipements trouvés !

Si vous considérez que tout ce matériel est (re)connu et va vous servir pour votre inventaire, vous pouvez l'importer en le sélectionnant et en indiquant le **type** d'équipement, sinon GLPI refusera de vous les importer (normal car il ne sera pas où le ranger). 
Une fois les équipements _typés_, re-sélectionnez-les puis faites **Actions** et **Import**.

Si maintenant vous allez voir dans **Ordinateurs** (ou autre en fonction de ce que vous avez importé), vos équipements sont bien là avec déjà pas mal de données pré-remplies. 
Vous pouvez d'ailleurs ensuite dire à GLPI de protéger les données déjà importées et de ne plus y toucher (regardez dans les `verrous (champs)`) !

Félicitations, vous avez fait le plus gros du boulot !

# Automatiser une tâche

Évidemment, tout ça est très bien, mais lancer l'agent à la main sur GLPI et sur Shinken, ce n'est pas très pratique... Il faut donc automatiser le processus. 
Si vous avez fouillé dans tout le Web comme moi à la recherche de tutoriel sur ce plugin, vous êtes certainement tombé sur ce problème. 
En fait, il y a une solution qui se trouve sur le [wiki officiel](http://wiki.glpi-project.org/doku.php?id=fr:config:crontab) mais qui n'est pas forcément facile à trouver.

On va donc définir 2 choses :

* Une tâche `cron` pour lancer le pseudo-cron de GLPI.
* Une tâche `cron` pour l'agent fusioninventory.

> Dans une précédente version de fusioninventory, il me semble que le mode Démon ne marchait pas très bien. C'est donc pour ça que j'ai configuré l'agent sous `cron`. 
Pour lancer l'agent en démon, il suffit juste de lui rajouter l'option `-d`. Vous pouvez rajouter `--debug` pour qu'il soit plus parlant.

## Tâche Cron pour FusionInventory

Pour configurer l'agent fusioninventory sous cron, il faut utiliser le cron de `root` ou un autre si vous avez un utilisateur dédié, mais au moins qui ai les droit pour le lancer. 
Ensuite, il faut simplement lui indiquer le temps voulu dans cron (`sudo crontab -e`) :

```conf
# Adaptez le chemin de fusioninventory à ce job !
# La tâche s'exécutera ici tous les jours de chaque mois à 3h05.
5 3 * * * /usr/local/bin/fusioninventory-agent
```

Enregistrez et quittez.

## Tache Cron pour GLPI

Comme dit sur la documentation citée ci-dessus, il suffit d'éditer le cron de **l'utilisateur du serveur web** qui fait tourner GLPI ! Si vous le faites sur un autre, cela ne marchera pas. 
A noter qu'il vous faut `php5-cli` comme prérequis :

```bash
sudo apt-get install php5-cli
```

Une fois cela fait lancer l'éditeur de cron pour votre utilisateur :

```bash
sudo crontab -u www-data -e
```

Puis ajouter la ligne suivante à la fin du fichier :

```conf
# Le chemin de GLPI doit correspondre à celui de votre installation !
*/1 * * * * /usr/bin/php5 /var/www/glpi/front/cron.php &>/dev/null
```

Enregistrez et quittez **crontab**. 

Cette tâche va toutes les minutes accéder au fichier `cron.php`. Vous pouvez définir une autre fréquence en fonction de votre installation, libre à vous de voir ce qui est le plus productif. 
Le job va donc créer artificiellement une activité dans GLPI ce qui aura pour conséquence de déclencher les actions automatiques de GLPI, dont notre `taskscheduler`.

# Conclusion

Si vous êtes patient, vous pouvez attendre 3h du matin pour voir si ça marche... sinon vous pouvez réduire le temps du job de l'agent, pour déjà tester votre processus. 
Vous pourrez suivre ensuite votre inventaire sur GLPI dans le **Monitoring / Logs** de Fusioninventory. 
Il va passer en _Préparé_, puis ensuite suivra les étapes jusqu'à la fin. 
Vous pouvez aussi vérifier la bonne marche de ces jobs via les logs de Cron :

```bash
grep CRON /var/log/syslog
```

Maintenant vous avez un moyen d'automatiser l'inventaire de votre parc de façon simple et surtout vous êtes sûr que vos données restent à jour.


