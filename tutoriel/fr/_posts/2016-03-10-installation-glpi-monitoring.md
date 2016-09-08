---
layout: post
title: Installer GLPI-Monitoring
lang: fr
ref: glpimonitoring
modified:
description: comment installer glpi-monitoring
tags: [tutoriel, glpi, shinken, monitoring]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-03-10T10:25:10+01:00
---

# Introduction

Ce tutoriel est destiné aux personnes qui possèdent un serveur [GLPI](http://glpi-project.org/) et un serveur [Shinken](http://www.shinken-monitoring.org/) ; et qui veulent monitorer leurs serveurs et afficher directement les données de Shinken dans GLPI. Vous devez avoir déjà des serveurs/ ordinateurs dans GLPI. Si vous n'avez pas ces serveurs, je vous conseille de suivre les tutoriels suivants :

* [Installer Shinken](/2016/03/shinken-installation)
* [Installer GLPI](/2016/03/glpi-installation)

Pour déjà avoir des données dans GLPI, vous devrez les remplir à la main ou bien utiliser un outils automatisé tel que [FusionInventory](/2016/03/fusion-inventory). Il vous permettra d'avoir des hôtes à monitorer.

Les configurations qui vont suivre ont été faites sur un serveur Shinken 2.4.2 et un serveur GLPI 0.90.1. Il est donc conseillé d'avoir des serveurs quasiment semblables. Toutefois, la plupart des configurations sont censées être quasiment identiques, hormis les versions des plugins utilisés.

> **Note :** Vous ne pourrez pas remonter des hôtes déjà créés dans les fichiers de Shinken vers GLPI !

# Prérequis

Pour suivre ce tutoriel, vous devrez avoir les prérequis suivants :

* Un serveur GLPI avec un accès `root` ou un compte qui possède les droits suffisants sur l'installation GLPI.
* Un serveur Shinken avec un accès `root` !
* Des enregistrements DNS qui permettent aux 2 serveurs cités ci-dessus de communiquer.

Le mieux est d'aussi avoir un minimum d'hôtes/serveurs dans GLPI pour que cela soit intéressant à monitorer.

# Plugins GLPI

Pour le moment nous allons déjà installer les plugins requis par GLPI pour pouvoir communiquer avec Shinken. GLPI va avoir besoin de 2 plugins :

* [glpi_Monitoring](https://github.com/ddurieux/glpi_monitoring) : ce plugin servira à afficher les différents serveurs que vous monitorer dans Shinken.
* [web_services](https://forge.glpi-project.org/projects/webservices) : ce plugin permettra aux deux serveurs de pouvoir communiquer.

Pour que cela soit pratique, je vous conseille de faire un dossier `plugin-glpi` dans votre `HOME` pour pouvoir y revenir plus tard lors de futurs mises à jour. Je vous conseille aussi d'utiliser [Git](https://git-scm.com/) pour récupérer vos plugins plus facilement.

## Téléchargement de glpi_monitoring

Pour récupérer ce plugin, vous n'avez qu'à cloner le dépôt et basculer sur le `tag` qui vous intéresse (ici `0.90+1.0`) :

```bash
git clone https://github.com/ddurieux/glpi_monitoring.git
cd glpi_monitoring
git checkout 0.90+1.0
cd ..
mv glpi_monitoring/ monitoring
sudo cp -R monitoring/ /var/www/glpi/plugins/
sudo chown -R www-data:www-data /var/www/glpi/plugins/monitoring
```

Voilà votre plugin "monitoring" est prêt.

Il est possible par la suite que GLPI vous fasse l'erreur suivante :

`PHP Fatal error:  Call to undefined function curl_init()`

Cela veut dire qu'il vous manque l'extention `curl` de `php5`. Dans ce cas :

```bash
sudo apt-get install php5-curl
```

Cela résoudra le problème.

## Téléchargement de web_services

Ce plugin est disponible uniquement sur la forge GLPI :

```bash
wget https://forge.glpi-project.org/attachments/download/2099/glpi-webservices-1.6.0.tar.gz
tar xzvf glpi-webservices-1.6.0.tar.gz
sudo cp -R webservices/ /var/www/glpi/plugins/
sudo chown -R www-data:www-data /var/www/glpi/plugins/webservices/
```

Web_services est prêt.

> **Note :** GLPI va reconnaître le plugin grâce à son nom de dossier et d'autres fichiers. faites bien attention à ne pas modifier le nom des dossiers de vos plugins sauf si GLPI vous le demande !

## Activer les plugins

Rendez-vous maintenant sur votre interface GLPI et connectez-vous. 

Si vous allez dans le menu **Configuration => Plugins** vous devriez voir apparaître vos deux plugins fraichement installés. Il est possible que le module **Services Web** vous affiche un message : `Installation PHP incorrecte. Nécessite le module xmlrpc` ! C'est normal si votre serveur n'a pas l'extention `xmlrpc` d'installé mais il est nécessaire pour le bon fonctionnement de celui-ci.

```bash
sudo apt-get install -y php5-xmlrpc
sudo service apache2 restart
```

Rafraichissez votre page, vous ne devriez plus avoir d'erreur.

Maintenant, vous pouvez cliquer sur les boutons **Installer** de chaque plugin et ensuite sur le bouton **Activer**. Ça y est vos plugins sont prêt à être utilisés. Si vous vous rendez sur **Plugins => Monitoring** vous ne verrez qu'un page blanche. C'est normal pour le moment il n'a rien à afficher mais nous allons lui donner de quoi travailler.

# Ajouter les utilisateurs nécessaires dans GLPI

Vous allez devoir créer un utilisateur dans MySQL et dans GLPI afin qu'ils puissent se connecter au plugin _Web Services_ pour récupérer sa configuration.

* Pour GLPI : allez dans **Administration => Utilisateurs** et ajoutez un compte `shinken`.

* Pour MySQL suivez les étapes ci-dessous :

Connectez vous à MySQL sur la base `glpi` :

```bash
mysql -u root -p glpi
```

Et tapez les commandes suivantes (en changeant le mot de passe évidemment) :

```mysql
GRANT select,update ON glpi_plugin_monitoring_services TO shinkenbroker IDENTIFIED BY 'password';
GRANT insert ON glpi_plugin_monitoring_serviceevents TO shinkenbroker;
GRANT select,update ON glpi_plugin_monitoring_servicescatalogs TO shinkenbroker;
```

> **Attention :** si votre serveur Shinken et votre serveur GLPI ne sont pas sur le même serveur, il faudra définir l'utilisateur avec son nom de machine : `'shinkenbroker'@'ip_serveur_shinken'` !

Tapez `CTRL-D` ou `exit` pour sortir de MySQL.

Voilà vos deux utilisateurs sont prêts.

# Configurer Web Services

Rendez vous sur l'interface GLPI et allez dans **Configuration => Webservices** et ajoutez un service avec les paramètres suivants :

* Nom : Shinken
* Services actifs : Oui
* Activez la compression : Non (Pas testé cette fonctionnalité jusqu'à présent)
* Tracer les connexions : Non (activez le si vous voulez garder une trace des connexions)
* Debug : Non (Activez-le en cas de debugging)
* Motif SQL des services : `.*` 
* Plage d'adresse IPv4 : mettez l'adresse de votre serveur Shinken dans les 2 cases.
* Adresse IPv6 : au cas où vous avez l'IPv6 d'activée.
* Identifiant utilisateur : laissez vide dans ce cas.
* Mot de passe : laissez vide dans ce cas.

Enfin cliquez sur le bouton **Ajouter** pour valider votre configuration. Rien de plus simple.

# Modules Shinken

Passons maintenant à la configuration des modules de Shinken. Vous allez devoir paramétrer les démons **Arbiter** et **Broker** pour qu'ils puissent se connecter à GLPI et sa Base de Données, et installer les modules requis en conséquence.

En tant qu'utilisateur `shinken` sur le serveur Shinken :

```bash
shinken install import-glpi
shinken install glpidb
shinken install ws-arbiter
```

## Configuration d'import-glpi

Vous devez avoir un fichier de configuration `import-glpi.cfg` dans votre dossier `modules`, ouvrez-le et éditez-le selon votre configuration :

```conf
define module {
    module_name     import-glpi
    module_type     import-glpi
    # L'URI de Web service pour GLPI
    uri             http://ip_serveur/glpi/plugins/webservices/xmlrpc.php
    # Si vous avez un nom de domaine, faites pointer sur la bonne adresse :
    # uri             http://domaine.com/plugins/webservices/xmlrpc.php
    # Les identifiants que vous avez créés dans GLPI juste avant
    login_name      shinken
    login_password  password
}
```

Enregistrez et quittez.

Il faut ensuite que vous ajoutiez ce module dans l'Arbiter (`vi arbiters/arbiter-master.cfg`) :

```conf
[...]
address     0.0.0.0     # Changez localhost en 0.0.0.0 affichera l'Etat du système dans GLPI.
modules     import-glpi
[...]
```

Enregistrez et quittez ce fichier. Voilà, Shinken doit pouvoir se connecter à GLPI.

## Configuration de glpidb

Vous devez tout d'abord modifier la configuration du module (`vi modules/glpidb.cfg`) :

```conf
define module {
    module_name     glpidb
    module_type     glpidb
    host            ip_serveur     # Le nom du serveur GLPI ou son IP
    port            3306
    database        glpi           # Le nom de la base GLPI
    user            shinkenbroker  # L'utilisateur créé dans MySQL
    password        password        # le mot de passe de l'utilisateur
[...]
}
```

Enregistrez ce fichier et quittez. Maintenant rajoutez le module dans votre Broker (`vi brokers/broker-master.cfg`) :

```conf
[...]
modules    webui2,glpidb
[...]
```

Enregistrez et quittez ce fichier. Shinken a maintenant ce qu'il lui faut pour se connecter à la base de données de GLPI.

## Configuration de ws-arbiter

Pour ce module il n'y aucune modification particulière à effectuer. Toutefois si dans GLPI vous avez défini un mot de passe et un identifiant dans le plugin `Webservices` vous devrez avoir les mêmes données dans le fichier de configuration (`modules/ws_arbiter.cfg`).

Vous devez quand même l'indiquer dans l'Arbiter de Shinken (`arbiters/arbiter-master.cfg`) :

```conf
[...]
modules     import-glpi,ws-arbiter
[...]
```

Enregistrez vos modifications et quittez.

# Vérifier si tout fonctionne

Maintenant vous avez normalement toutes vos configurations de prêtes. Il ne vous reste plus qu'à redémarrer Shinken (et GLPI si besoin) :

Sur le serveur Shinken :

```bash
sudo service shinken restart
```

Sur celui de GLPI

```bash
sudo service apache2 restart
```

Et normalement vous devriez voir la fenêtre suivante si vous allez sur l'interface de GLPI :

<figure>
    <img src="{{ site.url }}/images/glpi/glpi_monitoring.png" alt="">
    <figcaption>GLPI Monitoring</figcaption>
</figure>

Vous devriez voir une pastille verte en face d'`État du système` vous indiquant que tous les démons de Shinken fonctionne ainsi que le `ping`. Vous pouvez voir aussi que vous pouvez redémarrer Shinken à partir de l'interface de GLPI. Si vous redémarrez Shinken à partir d'ici une notification GLPI s'affichera  : `La communication avec Shinken fonctionne parfaitement`.

## Vérifiez les logs

Si vous rencontrez des soucis, n'oubliez pas de vérifier vos logs ! 

Sur le serveur Shinken :

```bash
sudo tail -f /var/log/shinken/*.log | grep ERROR
```

Cela devrait déjà permettre de repérer les erreurs éventuelles. Il est aussi possible de tester votre connexion `xmlrpc` via la commande suivante en tant qu'utilisateur **shinken** :

```bash
shinken-arbiter -v -c /etc/shinken/shinken.cfg
```

Pour GLPI, il suffit de regarder les logs d'apache2 et de GLPI :

```bash
sudo tail -f /var/log/apache2/*error.log
# ou bien dans le dossier files de GLPI
sudo tail -f /var/www/glpi/files/_log/*.log
```

Si tout fonctionne, il ne vous restera plus qu'à configurer vos commandes, vos composants et des catalogues de composants (ou de services). Ils seront ensuite disponibles dans les différents menus (État des machines, Ressources, Métriques, etc...).

# Configurer un catalogue

## Commandes

**Note :** les plugins nagios (et donc vos commndes) ont été normalement installés, précédemment, dans le tutoriel de [Shinken](/2016/03/shinken-installation). 

Les commandes définies dans **glpi-monitoring** ne sont pas forcément configurées pour votre installation. Vous pouvez ne pas avoir les plugins définis dans le bon dossier et votre variable `$NAGIOSPLUGINSDIR$` doit être mis à jour en fonction de votre configuration.

Pour rappel vous pouvez configurer vos `$PATH` dans : `SHINKEN_DIR/resource.d/paths.cfg` (sur votre serveur Shinken).

Il est aussi mieux de tester la commande en local avant pour voir si elle fonctionne bien. Par exemple :

```bash
user@shinken:~$./check_snmp_storage.pl -H 127.0.0.1 -C public -m / -w 80% -c 90% -G
/dev: 0%used(0GB/0GB) /: 20%used(11GB/57GB) (<80%) : OK
```

## Composants

Une fois une commande définie, il faut que vous associez un `composant` à celle-ci afin de pouvoir l'appliquer par la suite. Remplissez au minimum les champs suivant :

* Donnez un nom
* Sélectionnez la commande que vous souhaitez
* Sélectionnez un gabarit de graphique. La plupart des gabarits sont déjà bien défini. Si il vous manque un gabarit de graphique, créez-en un :
  * Il suffit de mettre votre commande dans le champ `Exemple de perfdata pour ce contrôle` et de cliquer sur **Sauvegarder**. Le plugin vous génèrera tout seul le gabarit.
* Définissez un Contrôle : cela sert à savoir combien de fois shinken va tenter cette commande avant de changer d'état.
* Contrôle actif => Oui
* Contrôle passif => Oui
* Période de contrôle : sélectionnez une période.

Puis sauvegardez !

## Catalogue de Composants

Il ne vous reste plus qu'à définir un catalogue de composant pour tout ça. Le catalogue sera à définir selon vos souhaits : plusieurs hôtes par catalogues ou non, plusieurs composants, etc... Libre à vous de voir comment vous voulez ensuite monitorer vos hôtes. Toutefois le processus, ressemblera à peu près à ce qui suit :

* Donnez lui un nom
* Rajoutez un (ou plusieurs) composant
* Rajoutez un hôte : vous pouvez définir une règle pour l'importer de façon dynamique ou choisir un hôte statique.
* Rajoutez un contact : vous pouvez aussi rajouter un contact qui sera prévenu si l'_état_ de vos composants change (UP, DOWN, WARNING, etc...).

Une fois votre catalogue défini, Shinken devrait normalement redémarrer automatiquement lorsque vous cliquerez sur **Ajouter** et donc rajouter le catalogue créé.

# Conclusion

Glpi-Monitoring n'est pas forcément simple à configurer et installer ; notamment l'installation complète de tous ces serveurs.

Prenez le temps de bien vérifier vos configurations et vos logs afin de vous assurez qu'il n'ya pas des erreurs qui trainent. N'hésitez pas non plus à contacter les équipes de développement sur les forums ou les canaux IRC.

Gardez aussi en tête que nous sommes pour la plupart bénévoles pour tous ces software et que nous ne pouvons pas toujours répondre rapidement.
