---
layout: post
title: Installer Redmine 3.0
lang: fr
ref: redmine
modified:
description: Comment installer Redmine
tags: [tutoriel, redmine]
image:
  feature:
  credit:
  creditlink:
comments: true
share:
date: 2016-03-02T12:00:05+01:00
---

# Introduction

Dans les entreprises, il y a toujours plusieurs projets qui demandent un suivi, une gestion de tickets, de documents, de temps passé, etc. Certains logiciels payant font très bien ce travail mais ont quand même un certain coût. Pour pallier à cela, il existe des gestionnaires de projets Open Source, dont [Redmine](http://www.redmine.org/) fait parti.

Redmine est une véritable usine à gaz, permettant de réaliser beaucoup de choses. Il a l'avantage de pouvoir discuter avec beaucoup d'autres serveurs (tel que Gitlab ou Jenkins) et d'être de base très complet. Des plugins sont aussi disponibles et il est possible d'en créer soi-même.

# Prérequis du Tutoriel

Nous allons donc voir comment installer Redmine dans sa dernière version stable (actuellement 3.2). De l'installation du serveur de base jusqu'à un serveur de production, avec son serveur Web et son service.

Pour cela il va falloir vous assurer d'avoir les prérequis suivants :

* Une machine Ubuntu (ou Debian-like) avec un accès **root**
* Le programme curl d'installé (`sudo apt-get install curl`)
* Des connaissances en Linux et de la patience...

# Prérequis de Redmine

Redmine a besoin de pas mal de choses pour pouvoir fonctionner. Il utilise Ruby et MySQL (ou une autre base de données), ainsi que les programmes Gem, Bundler et plusieurs autres dépendances. Il est toujours important de consulter les [prérequis de Redmine](http://www.redmine.org/projects/redmine/wiki/RedmineInstall#Requirements) avant de se lancer dans une installation.

A ce jour (Mars 2016), voici ce que préconise la documentation :

| Version de Redmine | Versions de Ruby supportées    | Version de Rails utilisée |
|:------------------:|:------------------------------:|:-------------------------:|
| Version en cours   | ruby 1.9.3, 2.0.0, 2.1, 2.2 | Rails 4.2                 |
| Redmine 3.0        | ruby 1.9.3, 2.0.0, 2.1, 2.2 | Rails 4.2                 |
| Redmine 2.6        | ruby 1.8.7, 1.9.2, 1.9.3, <br>2.0.0, 2.1, 2.2, jruby-1.7.6 | Rails 3.2 |

Base de Données Supportées : MySQL 5.0 ou plus, PostgreSQL 8.2 ou plus, Microsoft SQL Server 2008 (Redmine 2.X) ou 2012 (Redmine 3.X) et _SQLite 3_.

Comme vous pouvez le voir, Redmine supporte pas mal de version de Ruby différentes et de base SQL. Ce qui est pratique sans l'être. Ruby et Rails sont assez sensibles dès qu'on mélange plusieurs versions ou leurs librairies, certaines versions de Ruby ont des bugs connus, etc...

Dans ce tutoriel, nous allons donc utiliser Ruby 2.1 (qui apparemment ne pose aucun souci) et MySQL pour installer Redmine en 3.X.

## Créer un utilisateur dédié

Pour des raisons de bonnes pratiques, il est toujours mieux de créer un utilisateur qui va s'occuper du service créé. Personnellement, j'ai créé un utilisateur qui se nomme `redmine` pour garder une logique mais ce n'est pas une obligation.

J'ai aussi choisi de mettre son _HOME_ dans un répertoire différent du _HOME_ habituel (`/home/`) mais ce n'est pas obligatoire non plus.

```bash
sudo mkdir /usr/local/home
sudo useradd -s /bin/bash -m -d /usr/local/home -g root redmine
```

Puis connectez vous avec votre utilisateur fraichement créé :

```bash
sudo su - redmine
```

Dans la suite de ce tutoriel vous n'aurez normalement pas à rechanger d'utilisateur.

## Installer Ruby

Pour installer Ruby, on pourrait passer par les paquets proposé par votre distribution. Mais celui-ci n'est pas forcément à jour et ne proposera peut-être pas de version assez récente pour installer Redmine 3.X. Je vous conseille donc de passer par [RVM](https://rvm.io/).

L'avantage avec RVM, c'est qu'il vous permet de gérer facilement vos versions de Ruby et surtout de pouvoir les mettre à jour en peu de temps.

Pour installer RVM, tapez la commande suivante (en tant qu'utilisateur **redmine** !) :

```
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
```

> **Note :** Il est possible que l'obtention de RVM diffère sur Debian ou sur d'autres distribution. Pour Ubuntu j'ai du forcer le port 80 et pointer sur `keyserver.ubuntu`. La documentation officielle préconise de pointer sur le serveur de clé : `hkp://keys.gnupg.net`.

Normalement si tout se passe bien, vous devriez avoir la sortie suivante :

```
Downloading https://github.com/rvm/rvm/archive/1.26.11.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.26.11/1.26.11.tar.gz.asc
gpg: Signature faite le lun. 30 mars 2015 23:52:13 CEST avec la clef RSA d'identifiant BF04FF17
gpg: Bonne signature de « Michal Papis (RVM signing) <mpapis@gmail.com> »
gpg: Attention : cette clef n'est pas certifiée avec une signature de confiance.
gpg:          Rien n'indique que la signature appartient à son propriétaire.
Empreinte de clef principale : 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
     Empreinte de la sous-clef : 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/home/redmine/.rvm/archives/rvm-1.26.11.tgz'

Installing RVM to /usr/local/home/redmine/.rvm/
    Adding rvm PATH line to /usr/local/home/redmine/.profile /usr/local/home/redmine/.mkshrc /usr/local/home/redmine/.bashrc /usr/local/home/redmine/.zshrc.
    Adding rvm loading line to /usr/local/home/redmine/.profile /usr/local/home/redmine/.bash_profile /usr/local/home/redmine/.zlogin.
Installation of RVM in /usr/local/home/redmine/.rvm/ is almost complete:

  * To start using RVM you need to run `source /usr/local/home/redmine/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.

# redmine,
#
#   Thank you for using RVM!
#   We sincerely hope that RVM helps to make your life easier and more enjoyable!!!
#
# ~Wayne, Michal & team.

In case of problems: http://rvm.io/help and https://twitter.com/rvm_io
```

Voilà rvm est installé. Comme dit dans l'installation, pour commencer à l'utiliser vous devez taper la commande suivante :

```bash
source /usr/local/home/redmine/.rvm/scripts/rvm
```

Maintenant que vous avez rvm d'installé, vous pouvez voir les différentes versions de Ruby disponibles en tapant `rvm list known`. Il ne reste plus qu'à lui indiquer quel version de ruby vous souhaitez installer grâce à la commande :

```bash
rvm install 2.1.5
```

Le programme vous demandera le mot de passe `root` pour pouvoir continuer l'installation et installera tout seul les dépendances requises (gcc, g++, make, zlib, etc...). Si tout se passe correctement, vous devriez avoir Ruby d'installé. Vous pouvez vérifier en tapant `ruby --version`.

## Installer MySQL

Pour installer MySQL, c'est beaucoup plus simple évidemment. Une simple installation par les dépôts sera suffisante, ainsi qu'une librairie en plus pour l'adaptateur `mysql2`.

```bash
sudo apt-get update
sudo apt-get install mysql-server libmysqlclient-dev
```

Vous devrez certainement créer un mot de passe pour le compte `root` de MySQL. Retenez le bien afin de pouvoir y accéder par la suite.

Félicitations ! Vous avez tous les prérequis nécessaires pour pouvoir installer Redmine.

# Installation de Redmine

Maintenant que vous avez fait le plus gros du boulot, il ne vous reste plus qu'à télécharger l'archive qui vous intéresse. 

## Téléchargement de l'archive

Pour cela, rendez-vous sur la page des dernières [versions stables de Redmine](http://www.redmine.org/projects/redmine/wiki/Download#Stable-releases) et télécharger l'archive qui vous intéresse. Décompressez-la dans le _HOME_ que vous avez choisi et renommez le dossier `redmine-X.X.X` en `redmine`.

Dans notre exemple, cela donne :

```bash
cd ~; wget http://www.redmine.org/releases/redmine-3.2.0.tar.gz
tar -xzvf redmine-3.2.0.tar.gz
mv redmine-3.2.0 redmine
```

## Configurer la Base SQL

Pour que Redmine puisse ranger et organiser toutes ses informations, il lui faut une base de données.

### Configurer SQL

Connectez vous à SQL avec votre compte `root` et tapez les lignes suivantes (en remplaçant `your_password` par un mot de passe sécurisé) :

```bash
mysql -u <utilisateur_root> -p
```

```sql
mysql> CREATE DATABASE redmine CHARACTER SET utf8;
mysql> CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'your_password';
mysql> GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```

Vous pouvez ensuite quitter MySQL en tapant `CTRL-D` ou `exit`.

> **Note :** encore une fois l'utilisateur SQL et sa base s'appellent _redmine_ mais ce n'est pas obligatoire. Libre à vous de les appeler comme vous voulez, tant que vous vous y retrouvez et que vous appliquez ensuite les modifications en conséquence dans les fichiers de configuration.

### Configurer SQL pour Redmine

Redmine fonctionne avec des fichiers de configuration portant l'extention `.yml`. Et ses créateurs sont sympas, ils vous ont mis des exemples partout. Ce qui nous intéresse maintenant c'est de dire à Redmine sur quelle Base de Données il doit travailler, on va donc copiez le fichier `conf/database.yml.example` :

```bash
cd ~/redmine
cp config/database.yml.example config/database.yml
```

Ouvrez le fichier copié (`vi config/database.yml`) et éditez-le en fonction de votre configuration. Dans notre exemple, cela devrait ressembler à ça :

```conf
# Des exemples pour PostgreSQL, SQLite3 et SQL Server peuvent être trouvés à la fin du fichier.
# L'indentation des lignes doivent être de 2 espaces (pas de tabulations).

production:
  adapter: mysql2           # l'adaptateur utilisé
  database: redmine         # le nom de votre Base
  host: localhost           # le nom d'hôte de l'utilisateur
  username: redmine         # le nom d'utilisateur SQL
  password: "your_password" # le mot de passe de l'utilisateur
  encoding: utf8            # l'encodage défini pour la Base
```

Personnellement j'ai effacé les autres configurations qui ne m'étaient pas utiles, mais vous pouvez les laisser pour pouvoir les consulter ultérieurement.

## Installer les dépendances

Maintenant Redmine va avoir besoin de Bundler pour fonctionner et gérer les dépendances de ses Gems. Pour installer Bundler :

```bash
cd ~/redmine
gem install bundler
```

Ensuite, grâce à Bundler vous allez pouvoir installer les Gems dont Redmine a besoin :

```bash
bundle install --without development test rmagick
```

Si tout se passe bien, vous devriez voir plusieurs lignes d'installation (des différentes gems) et à la fin les lignes suivantes :

```bash
Bundle complete! XX Gemfile dependencies, XX gems now installed.
Gems in the groups development, test and rmagick were not installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

> **Note :** comme vous avez pu le voir, nous avons dis à Bundler de ne pas installer de gems pour RMagick. Cette gem est optionnelle et sert à utiliser ImageMagick pour manipuler des PDF et des PNG pour l'export de données. Si vous souhaitez quand même l'installer, assurez-vous d'avoir les dépendances requises avant de lancer `bundle install`. Pour Ubuntu, vous devrez installer les paquets `imagemagick` et `libmagickwand-dev` au préalable.

## Générer une clé de codage

Cette étape va permettre à Rails d'avoir une clé pour encoder les cookies de vos sessions. Pour générer cette clé tapez la commande suivante :

```bash
bundle exec rake generate_secret_token
```

## Création du Schéma de la Base

Redmine va avoir ensuite besoin d'une structure pour sa base de données pour pouvoir fonctionner. Elle doit correspondre à votre environnement, ici l'environnement est _production_.

```
RAILS_ENV=production bundle exec rake db:migrate
```

Vous allez avoir pas mal de ligne d'exécution qui vont défiler :

```bash
== 1 Setup: migrating =========================================================
-- create_table("attachments", {:force=>true})
   -> 0.0031s
-- create_table("auth_sources", {:force=>true})
   -> 0.0025s
...
== 20151024082034 AddTokensUpdatedOn: migrating ===============================
-- add_column(:tokens, :updated_on, :timestamp)
   -> 0.0020s
== 20151024082034 AddTokensUpdatedOn: migrated (0.0028s) ======================

== 20151025072118 CreateCustomFieldEnumerations: migrating ====================
-- create_table(:custom_field_enumerations)
   -> 0.0014s
== 20151025072118 CreateCustomFieldEnumerations: migrated (0.0018s) ===========

== 20151031095005 AddProjectsDefaultVersionId: migrating ======================
-- column_exists?(:projects, :default_version_id, :integer)
   -> 0.0009s
-- add_column(:projects, :default_version_id, :integer, {:default=>nil})
   -> 0.0024s
== 20151031095005 AddProjectsDefaultVersionId: migrated (0.0042s) =============
```

Vous devrez ensuite insérer les données par défaut (ainsi que le language) dans votre Base :

```bash
RAILS_ENV=production REDMINE_LANG=fr bundle exec rake redmine:load_default_data
```

Cela devrait afficher la sortie suivante :

```bash
Default configuration data loaded.
```

Votre Base est donc à jour et contient tout ce qui faut pour que Redmine soit opérationnel.

## Permissions et Fichiers

Vous devrez ensuite faire en sorte que certains dossiers soient accessibles et leur donner les permissions suivantes :

```bash
mkdir -p tmp tmp/pdf public/plugin_assets
chmod -R 755 files log tmp public/plugin_assets
```

Si vous n'avez pas créé votre dossier redmine dans le _HOME_ de l'utilisateur ou que vous avez fait une installation différente, il faudra quand même vous assurer que l'utilisateur ai les droits sur les dossiers suivants : `files`, `log`, `tmp` et `public/plugin_assets`.

Félicitations ! Vous avez enfin une installation qui est prête !

# Tester l'installation

Pour tester votre serveur Redmine, il vous faudra maintenant exécuter la commande suivante :

```bash
bundle exec rails server webrick -e production
```

> **Attention**, cette commande ne sert qu'à tester l'installation et ne doit pas être utilisée en "production" ! Nous verrons par la suite comment monter le service Redmine.

Ouvrez votre navigateur et pointez sur l'adresse suivante : [http://localhost:3000/](http://localhost:3000/). Si vous n'avez qu'une console, téléchargez l'index de cette adresse (`wget http://localhost:3000/`).

> **Astuce :** si vous avez fait ce tutoriel sur une machine virtuelle qui n'a pas d'interface graphique (il y a de fortes chances que ce soit le cas !), vous pouvez faire en sorte que le localhost du serveur devienne le votre. Pour réussir ce _tour de magie_, il faut monter un tunnel SSH entre votre PC et le Serveur, avec une redirection de port (ici le port est _3000_). Voici un exemple : `ssh -L 3000:127.0.0.1:3000 login@ip_serveur`. Pointez ensuite sur l'adresse _http://localhost:3000_ avec votre navigateur et vous devriez arriver sur l'interface de Redmine !

Pour pouvoir se logger, il suffit d'utiliser les identifiants suivants :

* Login : admin
* Mot de Passe : admin

Je vous conseille bien évidemment de les changer par la suite.

Vous pouvez à tout moment arrêter le serveur en tapant `CTRL-C`.

# Créer le Service Redmine

Vous avez enfin un serveur Redmine opérationnel... Mais par contre il n'est accessible que sur la boucle locale du serveur (localhost) et implique d'aller sur le port 3000. De plus, il nous indique que l'on ne doit pas utiliser Redmine de cette manière en production. Bref, tout cela n'est pas très joli.

Nous allons donc créer un service à l'aide la gem _unicorn_ et un serveur Web grâce à Nginx.

## Installer Unicorn

Pour pouvoir utiliser _unicorn_, nous devons d'abord l'installer. Ne touchez pas au Gemfile de Redmine et utilisez plutôt le fichier `Gemfile.local` dédié à ce genre de manipulation. Ajoutez la gem _unicorn_ à ce fichier (`vi Gemfile.local`) :

```conf
gem 'unicorn'
```

Puis relancer Bundler afin d'installer la nouvelle gem :

```bash
bundle install --without development test rmagick
```

Vous devriez voir l'installation de nouvelles gems (_unicorn_ et ses dépendances).

## Configuration pour Unicorn

Unicorn va maintenant avoir besoin d'un fichier de configuration nommé `unicorn.rb`. Nous allons placer ce fichier dans le dossier `config` de Redmine (`vi config/unicorn.rb`) et y ajouter les lignes suivantes :

```ruby
worker_processes 4
working_directory "/usr/local/home/redmine/redmine"
listen "/var/run/redmine/redmine-unicorn.sock", :backlog => 64
timeout 30

pid "/var/run/redmine/unicorn.pid"

stderr_path "/var/log/redmine/unicorn.stderr.log"
stdout_path "/var/log/redmine/unicorn.stdout.log"
```

Créez maintenant les dossiers correspondants et les droits nécéssaires :

```bash
sudo mkdir /var/run/redmine
sudo chown redmine:redmine /var/run/redmine/
sudo mkdir /var/log/redmine/
sudo chown redmine:redmine /var/log/redmine/
touch /var/log/redmine/unicorn.stderr.log
touch /var/log/redmine/unicorn.stdout.log
```

Voilà, _unicorn_ est configuré. Nous allons à présent créer le service associé.

## Service Unicorn-Redmine

Pour créer un service pour Redmine, il va falloir le rajouter dans `/etc/init.d/`. Attention à bien vérifier les chemins que vous allez mettre dans ce fichier, une simple erreur dans un des chemins et le service ne marchera pas.

Heureusement il y aura toujours les logs définis à l'étape précédente pour vous aider à débugger en cas de problème !

Créez votre fichier (`sudo vi /etc/init.d/redmine_unicorn`) et ajoutez les lignes suivantes :

```bash
#!/bin/bash

### BEGIN INIT INFO
# Provides: unicorn
# Required-Start: $remote_fs $syslog
# Required-Stop: $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start daemon at boot time
# Description: Enable service provided by daemon.
### END INIT INFO

PATH=/usr/local/home/redmine/.rvm/gems/ruby-2.1.5/bin:/usr/local/home/redmine/.rvm/gems/ruby-2.1.5@global/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/home/redmine/.rvm/bin
BUNDLE=/usr/local/home/redmine/.rvm/gems/ruby-2.1.5/bin/bundle
UNICORN=/usr/local/home/redmine/.rvm/gems/ruby-2.1.5/bin/unicorn_rails
KILL=/bin/kill
APP_ROOT=/usr/local/home/redmine/redmine
PID=/var/run/redmine/unicorn.pid
OLD_PID=/var/run/redmine/unicorn.pid.oldbin
CONF=/usr/local/home/redmine/redmine/config/unicorn.rb
GEM_HOME=/usr/local/home/redmine/.rvm/gems/ruby-2.1.5
USER=redmine

[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"  # This loads RVM

sig () {
  test -s "$PID" && kill -$1 `cat $PID`
}

case "$1" in
  start)
    echo "Starting unicorn rails app ..."
    su - $USER -c "cd $APP_ROOT; $BUNDLE exec unicorn_rails -D -E production -c $CONF" &&
    echo "Unicorn rails app started!" ||
    echo "Unicorn rails app failed"
    ;;
  stop)
    echo "Stoping unicorn rails app ..."
    sig QUIT && exit 0
    echo "Not running"
    ;;
  restart)
    if [ -f $PID ];
    then
      echo "Unicorn PID $PID exists"
      /bin/kill -s USR2 `/bin/cat $PID`
      sleep 30
      echo "Old PID $OLD_PID"
      /bin/kill -9 `/bin/cat $OLD_PID`
    else
      echo "Unicorn rails app is not running. Lets start it up!"
      $0 start
    fi
    ;;
  status)
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    ;;
esac
```

Si vous avez suivi scrupuleusement tout ce tutoriel, vous n'aurez normalement aucune modification à faire sur ce fichier. Mais vérifiez quand même les différentes variables définies dans celui-ci et comme dit précédemment les différents chemins.

Donnez ensuite les droits sur ce fichier et activez le service au démarrage du serveur :

```bash
sudo chmod 755 /etc/init.d/redmine-unicorn
sudo update-rc.d redmine-unicorn defaults
```

Vous pouvez maintenant démarrer le service redmine-unicorn :

```bash
sudo service redmine-unicorn start
```

Si vous avez une erreur, n'hésitez pas à consulter vos logs (`tail -f /var/log/redmine/*`).

# Installation du serveur Web

Il ne reste plus qu'à installer et configurer un serveur virtuel pour notre serveur Redmine. Nous utiliserons Nginx comme serveur Web.

## Installation de Nginx

Pour installer Nginx, rien de plus simple :

```bash
sudo apt-get install nginx
```

## Configuration du serveur Web

Une fois installé, nous allons configurer le serveur Web. Pour éviter tout conflit avec le serveur par défaut, vous pouvez soit effacer le fichier `default` qui se trouve dans `/etc/nginx/sites-available/` ou bien commenter les lignes avec les ports 80 :

```nginx
server {
    #listen 80 default_server;
    #listen [::]:80 default_server ipv6only=on;
```

Créez votre fichier de configuration pour votre serveur virtuel (`vi /etc/nginx/sites-available/redmine`) et ajoutez les lignes ci-dessous :

```nginx
# Nomme et définit le socket
upstream redmine_unicorn {
    server unix:/var/run/redmine/redmine-unicorn.sock fail_timeout=0;
}
# Définit le serveur
server {
    listen 80 default deferred;
    client_max_body_size 4g;
   
    # Nom du serveur et son dossier root
    server_name redmine.local.fr;
    root /usr/local/home/redmine/redmine/public;

    keepalive_timeout 5;

    try_files $uri/index.html $uri.html $uri @app;

    # Configuration du proxy
    location @app {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header Host $http_host;

        proxy_redirect off;

        proxy_pass http://redmine_unicorn;
    }

    error_log /var/log/nginx/error.log debug;

    error_page 500 502 503 504 /500.html;
    location = /500.html {
        root /usr/local/home/redmine/redmine/public;
    }
}
```

Enregistrez et quittez. Maintenant votre serveur virtuel est configuré, vous n'avez plus qu'a relancer le service Nginx :

```bash
sudo service nginx restart
```

Félicitations ! Vous devriez pouvoir accéder à votre serveur de production via l'url suivante : [http://redmine.local.fr](http://redmine.local.fr).

## Quelques solutions aux erreurs courantes

Si vous n'avez que la page d'accueil de Nginx : 

* Vérifiez que vous avez commenté (ou supprimé) le serveur `default`.
* Vérifiez que votre fichier de configuration nginx `redmine` n'est pas erroné (il peut manquer un `;` en fin de ligne par exemple).

Si vous avez une page blanche ou d'erreur :

* Consultez les logs de nginx : `sudo tail -f /var/log/nginx/error.log`

N'oubliez pas non plus de **relancer Nginx** à chaque modification sur ses fichiers de configuration.

# Conclusion

Vous savez maintenant comment monter un serveur Redmine stable. Redmine possède ensuite de nombreuses configurations comme les droits, les dépendances entre projets, les différents tickets, les champs personnalisables, etc. Il possède aussi un catalogue de plugins bien fourni, dont un qui vous permettra de synchroniser les utilisateurs Redmine via votre Active Directory. Ou encore un plugin qui vous permettra de lier les tickets de Redmine aux commits de Git (ou d'un serveur Gitlab).
