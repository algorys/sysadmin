---
layout: post
title: Installation de Gilab Community
lang: fr
ref: gitlab
modified:
description: Tutoriel pour Gitlab, un github-like chez soi
tags: [tutoriel, git]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-03-21T14:50:07+01:00
---

# Introduction

[Gitlab](https://about.gitlab.com/) est un serveur similaire à [Github](https://github.com). Il permet d'administrer et de centraliser vos dépôts [Git](https://git-scm.com/) d'une manière pratique grâce à une interface Web, une gestion de tickets, une API et plein d'autres fonctionnalités par rapport à un simple dépôt `bare`. 

L'avantage avec un tel serveur c'est qu'il vous permet de voir l'état de vos dépôts depuis n'importe quel plateforme / poste et surtout il amène une gestion de groupe complète et des outils pour discuter avec d'autres serveurs (comme Jenkins par exemple).

Ce tutoriel permet d'installer Gitlab dans sa version `Community`. Il reprend le [tutoriel officiel](http://doc.gitlab.com/ce/install/installation.html#packages-dependencies) déjà très bien fait et ne fait qu'apporter un peu plus d'indices (et de français !).

# Prérequis

Pour déployer un tel serveur, je recommande d'avoir déjà une bonne connaissance de Git (ou d'un logiciel équivalent) afin d'en saisir l'intérêt et surtout son fonctionnement.

* Un serveur Linux avec les droits `root`. 

_Ce tutoriel a été réalisé sur un Ubuntu Server 14.04.4 LTS_

# Dépendances

Comme tout serveur, Gitlab a besoin de quelques dépendances pour fonctionner. Notamment ici pour que Ruby puisse bien compiler et avoir les gems qu'il lui faut. Assurez-vous d'avoir un serveur à jour et installez les dépendances suivantes :

```bash
sudo apt-get update
sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake nodejs
```

Si vous voulez utiliser [Kerberos](https://fr.wikipedia.org/wiki/Kerberos_(protocole)), installer la librairie suivante :

```bash
sudo apt-get install libkrb5-dev
```

## Installer Git

Il vous faut maintenant vous assurer d'avoir une version de Git suffisante. La documentation préconise une version de Git en `2.7.4` ou supérieure et souvent votre distribution n'aura pas une version suffisamment haute dans ses paquets (en général pour une question de sécurité ou les distribution préfère attendre... ou parfois ce n'est tout simplement pas mis à jour). Pour vérifier cela, lancer la commande suivante et regarder le résultat :

```bash
sudo apt-cache policy git-core
```

Dans mon cas, j'ai eu la sortie suivante :

```bash
git-core:
  Installé : (aucun)
  Candidat : 1:1.9.1-1ubuntu0.2
 Table de version :
     1:1.9.1-1ubuntu0.2 0
        500 http://fr.archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
        500 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packages
     1:1.9.1-1 0
        500 http://fr.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
```

La version proposée est `1.9.1` ce qui est beaucoup trop bas. Pour pallier à cela, nous allons rajouter le dépôt PPA de Git-Core :

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
```

Si vous refaites un la même commande (`sudo apt-cache policy git-core`) vous devriez maintenant avoir une version à jour :

```bash
git-core:
  Installé : (aucun)
  Candidat : 1:2.7.4-0ppa1~ubuntu14.04.1
 Table de version :
     1:2.7.4-0ppa1~ubuntu14.04.1 0
        500 http://ppa.launchpad.net/git-core/ppa/ubuntu/ trusty/main amd64 Packages
     1:1.9.1-1ubuntu0.2 0
        500 http://fr.archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
        500 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packages
     1:1.9.1-1 0
        500 http://fr.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
```

Vous pouvez donc maitenant installer `git-core` :

```bash
sudo apt-get install git-core
```

Si vous voulez vraiment être sûr d'avoir la version requise :

```bash
algorys@srv-gitlab:~$ git --version
git version 2.7.4
```

> **Note :** j'ai installé un serveur Gitlab avec un version de Git 1.9.1 et qui fonctionne très bien. Cela est donc faisable, mais pour des raisons de sécurité, il est préférable d'avoir la dernière version d'installée.

Voilà, vous avez maintenant la configuration requise pour pouvoir installer Gitlab. Passons maintenant à Ruby.

# Ruby

Pour fonctionner Gitlab aura besoin de Ruby dans sa version 2.1.x. Les versions de Ruby 2.2 et 2.3 ne sont actuellement pas supportées. Il est indiqué aussi que les gestionnaires de version pour Ruby ne sont pas non plus supportés et peuvent poser différents problèmes pour vos futurs `push` et `pull` sur le serveur. Nous allons donc installer Ruby _à l'ancienne_ !

Vérifiez que vous n'avez pas de version de ruby d'installée et si le cas se présetnte, désinstalllez-la :

```bash
sudo apt-get remove rubyX.x
```

Où `X.x` est votre version installée.

Puis téléchargez Ruby :

```bash
mkdir /tmp/ruby && cd /tmp/ruby
curl -O --progress https://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.8.tar.gz
echo 'c7e50159357afd87b13dc5eaf4ac486a70011149  ruby-2.1.8.tar.gz' | shasum -c - && tar xzf ruby-2.1.8.tar.gz
```

Puis compilez-le :

```bash
cd ruby-2.1.8
./configure --disable-install-rdoc
make
sudo make install
```

Cela va prendre un peu de temps, vous pouvez allez boire un café en attendant !

Une fois l'installation de Ruby terminée, vous allez pouvoir installer la Gem `Bundler` :

```bash
sudo gem install bundler --no-ri --no-rdoc
```

Voilà Ruby est installé et prêt à être utilisé.

# Go

Depuis Gitlab 8.0, les requêtes HTTP de Git sont effectuée par `gitlab-workhorse`. C'est une sorte de démon écrit en Go. Pour l'installer, nous aurons besoin du compilateur Go. Les commandes suivantessont pour un Linux en 64-bit. Ajustez l'archive en fonction de votre plateforme en vous rendant sur la [page de téléchargement de Go](https://golang.org/dl/) !

```bash
curl -O --progress https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
echo '43afe0c5017e502630b1aea4d44b8a7f059bf60d7f29dfd58db454d4e4e0ae53  go1.5.3.linux-amd64.tar.gz' | shasum -a256 -c -
sudo tar -C /usr/local -xzf go1.5.3.linux-amd64.tar.gz
sudo ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
# Effacez l'archive inutile
rm go1.5.3.linux-amd64.tar.gz
```

# Utilisateur Git

Vous allez maintenant créer un utilisateur dédié pour Gitlab, sans mot de passe. Par convention, on donne comme nom `git` :

```bash
sudo adduser --disabled-login --gecos 'Gitlab' git
```

Par la suite, vous verrez que beaucoup de choses vont se retrouver dans le répertoire `HOME` de cet utilsateur.

# La Base de Données

Pour les fans de MySQL, vous allez être déçus, Gitlab ne recommande pas l'utilisation de cette base. Il est toutefois possible de l'utiliser en vous rendant sur la [Documentation MySQL](http://doc.gitlab.com/ce/install/database_mysql.html) prévue à cette effet.

Gitlab recommande donc PostgreSQL comme base de données. Une version 9.1 minium est requise ! Installez-la si ce n'est pas déjà fait  :

```bash
sudo apt-get install -y postgresql postgresql-client libpq-dev
```

Puis définissez un utilisateur `git` pour Gitlab dans celle-ci :

```bash
sudo -u postgres psql -d template1 -c "CREATE USER git CREATEDB;"
```

Créez la base dédiée à la production de Gitlab  et donnez tous les privilèges à l'utilsateur `git`

```bash
sudo -u postgres psql -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"
```

Essayez de vous connecter avec les données précédemment rentrées :

```bash
sudo -u git -H psql -d gitlabhq_production
```

Vous devriez maintenant avoir quelque chose de similaire à cela :

```sql
psql (9.3.11)
Type "help" for help.

gitlabhq_production=> 
```

Tapez `\q` ou `CTRL-D` pour quittez la session PostgreSQL.

# Installer Redis

Redis est un système de gestion de base de données très puissant et est nécessaire pour faire fonctionner Gitlab. Une version `2.8` minimum est requise ! Sur Ubuntu 14 et Debian 8, la version est normalement à jour. Installez-le avec la commande suivante :

```bash
sudo apt-get install redis-server
```

> Si vous n'avez pas une version suffisante, il faudra rajouter les dépôts en suivant la [documentation Redis de Gitlab](http://doc.gitlab.com/ce/install/redis.html). Sinon, vous pouvez continuer.

Il faut maintenant donner des droits à l'utilisateur `git` en l'ajoutant dans le groupe `redis` :

```bash
sudo usermod -aG redis git
```

Voilà, `redis-server` est prêt !

# Gitlab

Enfin ! Nous allons nous occuper de Gitlab !

Allez dans le HOME de l'utilisateur `git` et cloner le dépôt :

```bash
cd /home/git
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 8-6-stable gitlab
```

Maintenant il va falloir configurer Gitlab pour correspondre à ce qu'on installé précédemment :

```
cd /home/git/gitlab
sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
```

Ouvrez le fichier `gitlab.yml` (`sudo -u git -H vi config/gitlab.yml`). Vous aurez, en tête de fichier, les modifications à faire pour l'ajuster à votre configuration. Voici ce que vous devez vérifier :

```yaml
# Par défaut la valeur est à `localhost`. Vous devrez modifier cela si vous avez un FQDN de prévu pour ce serveur.
host: localhost
# Choisissez un mail pour les mails provenant de ce serveur
email_from: example@example.com
# Indiquez le bon PATH pour git
git:
  bin_path: /usr/bin/git
```

Bien sûr il y aura d'autres choses à configurer ultérieuremnt comme les connexions avec des comptes LDAP par exemple.

Une fois cela fait, copiez le fichier suivant :

```bash
sudo -u git -H cp config/secrets.yml.example config/secrets.yml
sudo -u git -H chmod 0600 config/secrets.yml
```

Assurez vous que Gitlab puisse écrire dans les répertoires suivants :

```bash
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/
sudo chmod -R u+rwX builds/
sudo chmod -R u+rwX shared/artifacts/
```

Créez maitenant un dossier `public/uploads/` et assurez-vous que personne d'autre puisse y écrire :

```bash
sudo -u git -H mkdir public/uploads/
sudo chmod 0700 public/uploads
```

Copiez maintenant le fichier `unicorn.rb` :

```bash
sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
```

Puis en fonction du nombre de coeurs de votre serveur et sa RAM éditez le fichier `unicorn.rb` (`sudo -u git -H vi config/unicorn.rb`). Vous pouvez voir votre nombre de coeur en tapant `nproc` dans votre terminal. Un nombre de _worker_ pour 2GB de RAM sera par exemple de 3.

```yml
worker_processes 2
```

Enregistrez et quittez.

Copiez le fichier de configuration `rack_attack.rb`.

```bash
sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
```

Configurer les paramètres globaux de l'utilisateur `git`. Ce sera utilisé lorsque vous éditerez quelque chose via le navigateur web :

```bash
sudo -u git -H git config --global core.autocrlf input
```

Enfin, copiez le fichier de configuration pour Redis et éditez-le si vous avez modifié le PATH du _socket_ de Redis :

```bash
sudo -u git -H cp config/resque.yml.example config/resque.yml
```

```conf
# Changement à faire pour le path du socket
# production: unix:/var/run/redis/redis.sock
production: unix:/path/to/new/sock.sock
```

## Configurer la base de données pour Gitlab

Copiez le fichier d'exemple et éditez-le pour correspondre à votre installation :

```bash
sudo -u git cp config/database.yml.postgresql config/database.yml
sudo -u git -H vi config/database.yml
# Rajoutez l'utilisateur git en face de username.
```

Rendez ce fichier lisible uniquement par git :

```bash
sudo -u git -H chmod o-rwx config/database.yml
```

## Installer les Gems

Maintenant il va falloir installer les gems avec `bundler`. Assurez-vous que vous ayez une version supérieur ou égale à `1.5.2` (`bundle -v`) et lancez l'installation des gems :

```bash
# Pour une installation avec PostgreSQL. 
# Si vous avez choisi MySQL, remplacez "mysql" par "postgres"
sudo -u git -H bundle install --deployment --without development test mysql aws kerberos
```

Vous devriez avoir une sortie ressemblant à ça :

```bash
Fetching gem metadata from https://rubygems.org/...
Installing ...
[...]
Bundle complete! 173 Gemfile dependencies, 264 gems now installed.
Gems in the groups development, test, mysql, aws and kerberos were not installed.
Bundled gems are installed into ./vendor/bundle.
```

> **Note :** il est possible que selon votre connection certaines `gems` prennent du temps à s'installer. Soyez patient et n'interrompez pas l'installation. Si il y a la moindre erreur, `bundle` est en général assez verbeux pour vous indiquer quoi faire.

# Installer le Shell de Gitlab

Le _Shell de Gitlab_ est un logiciel de gestion de dépôt développé spécialement pour Gitlab. Pour l'installer lancez la tâche pour `gitlab-shell` :

```bash
# Modifiez l'URL de redis si besoin
sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production
```

Cela va cloner le dépôt correspondant dans `/home/git/gitlab-shell` et créer un répertoire `repositories` dans le _HOME_ de l'utilisateur `git`. Voici la sortie console :

```bash
WARNING: This version of GitLab depends on gitlab-shell 2.6.11, but you're running Unknown. Please update gitlab-shell.
Missing `db_key_base` for 'production' environment. The secrets will be generated and stored in `config/secrets.yml`
Clonage dans '/home/git/gitlab-shell'...
remote: Counting objects: 2536, done.
remote: Compressing objects: 100% (879/879), done.
remote: Total 2536 (delta 1596), reused 2495 (delta 1568)
Réception d'objets: 100% (2536/2536), 357.72 KiB | 0 bytes/s, fait.
Résolution des deltas: 100% (1596/1596), fait.
Vérification de la connectivité... fait.
HEAD est maintenant à bceed73 Update CHANGELOG for 2.6.11
mkdir -p /home/git/repositories/: OK
mkdir -p /home/git/.ssh: OK
chmod 700 /home/git/.ssh: OK
touch /home/git/.ssh/authorized_keys: OK
chmod 600 /home/git/.ssh/authorized_keys: OK
chmod ug+rwX,o-rwx /home/git/repositories/: OK
```

Vérifiez ensuite le fichier de configuration (`sudo -u git -H vi /home/git/gitlab-shell/config.yml`). Il doit correspondre à votre installation, notamment si vous comptez utiliser un DNS et un FQDN :

```conf
gitlab_url: http://gitlab.local.fr/
```

Maintenant vous allez pouvoir installer `gitlab-workhorse`.

# Installer Gitlab-Workhorse

Rendez-vous dans la racine du _HOME_ de l'utilisateur `git` et clonez le dépôt. Enfin on bascule sur le bon `tag`.

```bash
cd /home/git
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-workhorse.git
cd gitlab-workhorse
sudo -u git -H git checkout v0.7.1
sudo -u git -H make
```

# Initialiser la base de données

Maintenant, nous pouvons initialiser la base de données. Normalement, ce tutoriel suit à peu de chose près la documentation officielle de Gitlab. Seulement personnellement j'ai rencontré l'erreur suivante lors de la commande `gitlab:setup` :

```bash
This will create the necessary database tables and seed the database.
You will lose any previous data stored in the database.
Do you want to continue (yes/no)? yes

gitlabhq_production already exists
-- enable_extension("plpgsql")
   -> 0.0238s
   -- enable_extension("pg_trgm")
   rake aborted!
   ActiveRecord::StatementInvalid: PG::UndefinedFile: ERROR:  could not open extension control file "/usr/share/postgresql/9.3/extension/pg_trgm.control": Aucun fichier ou dossier de ce type
   : CREATE EXTENSION IF NOT EXISTS "pg_trgm"
```

Il suffit d'installer ce qu'il faut :

```bash
# Adaptez à la version de votre PostgreSQL
sudo apt-get install -y postgresql-contrib-9.3
```

Initialisez la base de données :

```bash
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
# Répondez "yes" pour créer les tables
```

A la fin, vous devriez avoir la sortie suivante :

```bash
== Seed from /home/git/gitlab/db/fixtures/production/001_admin.rb
Administrator account created:

login:    root
password: You'll be prompted to create one on your first visit.
```

# Sécurisez votre fichier secrets.yml

Ce fichier garde les clés de chiffrement pour les sessions et les variables. Il est conseillé de faire une copie de ce fichier dans un endroit à part de votre installation Gitlab. Autrement dit dans un autre répertoire que `/home/git`.

# Installez le script de démarrage

Maintenant, il vous faut un scrit de démarrage pour pouvoir lancer, arrêter, relancer votre serveur Gitlab proprement et facilement. Heureusement tout est déjà prévu il suffit copier le bon fichier au bon endroit :

```bash
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
sudo update-rc.d gitlab defaults 21
```

Activez la rotation des logs :

```bash
sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
```

# Vérifier votre installation

Maintenant que la plupart des fichiers sont configurés et prêts, il faut vérifier que tout cela peut fonctionner. Pour cela, vous allez devoir lancer les tests suivants :

```bash
# Vérifier le status de l'application
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
# Sortie de la commande :
System information
System:     Ubuntu 14.04
Current User:   git
Using RVM:  no
Ruby Version:   2.1.8p440
Gem Version:    2.2.5
Bundler Version:1.11.2
Rake Version:   10.5.0
Sidekiq Version:4.0.1

GitLab information
Version:    8.6.0-rc4
Revision:   834dae6
Directory:  /home/git/gitlab
DB Adapter: postgresql
URL:        http://localhost
HTTP Clone URL: http://localhost/some-group/some-project.git
SSH Clone URL:  git@localhost:some-group/some-project.git
Using LDAP: no
Using Omniauth: no

GitLab Shell
Version:    2.6.11
Repositories:   /home/git/repositories/
Hooks:      /home/git/gitlab-shell/hooks/
Git:        /usr/bin/git
```

# Compilation des "assets"

Maintenant, il faut compiler les "assets" de Gitlab :

```bash
sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
```

Cela peut prendre un peu de temps, soyez patient...

# Démarrer Gitlab

Si toutes les commandes précédentes sont passées, cela veut dire que vous pouvez démarrer votre serveur Gitlab !

```bash
sudo service gitlab start
# Sortie
Starting GitLab Unicorn
Starting GitLab Sidekiq
Starting gitlab-workhorse

The GitLab Unicorn web server with pid 24274 is running.
The GitLab Sidekiq job dispatcher with pid 24320 is running.
The gitlab-workhorse with pid 24302 is running.
GitLab and all its components are up and running.
```

# L'interface Web avec Nginx

Voilà, votre serveur Gitlab tourne ! Félicitations ! Il ne reste plus que l'interface web à configurer.

Nous allons desservir Gitlan avec Nginx, qui est de base, le serveur Web officiellement supporté par Gitlab. Installez Nginx sur votre serveur :

```bash
sudo apt-get install -y nginx
```

## Configurer le site

Comme depuis le début, vous avez déjà un fichier d'example à copier et vous n'avez qu'à activer le site avec un lien symbolique :

```bash
sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab
sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab
```

Configurer le fichier pour l'adapter à votre configuration (`sudo vi /etc/nginx/sites-enabled/gitlab`) :

```conf
# Vous devrez certainement commenter les deux lignes suivantes pour que la configuration passe :
listen 0.0.0.0:80 default_server;
listen [::]:80 default_server;
# Sinon, supprimer le site par défaut de Nginx : sudo rm -f /etc/nginx/sites-enabled/default

# Rajouter le FQDN de votre installation
server_name gitlab.local.fr;
```

## Tester votre configuration

Validez votre configuration Nginx avec la commande suivante :

```bash
sudo nginx -t
```

## Redémarrez le serveur

Si votre configuration est correcte, il faut redémarrer Nginx :

```bash
sudo service nginx restart
```

# Finalisation

Vous pouvez re-checker votre installation en relançant la commande suivante :

```bash
sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```

## Erreurs possibles

Lors de cette commandes, vous aurez peut-être des problèmes d'accès ou des soucis avec l'API de Gitlab. Normalement, Gitlab vous dira ce que vous devez faire pour régler chaque souci. Vérifiez vos entrées dans les fichiers de configuration.

Voici un exemple d'erreur :

```bash
Check GitLab API access: FAILED. code: 404
gitlab-shell self-check failed
  Try fixing it:
  Make sure GitLab is running;
  Check the gitlab-shell configuration file:
  sudo -u git -H editor /home/git/gitlab-shell/config.yml
  Please fix the error above and rerun the checks.
```

Vérifiez que vous avez bien renseigné le FQDN dans `/home/git/gitlab-shell/config.yml` et dans Nginx. Regardez votre fichier host ! (la boucle locale était sur `127.0.1.1`, j'ai du passer en `127.0.0.1` afin de régler ce problème). Et vérifiez que votre DNS puisse être résolu ! (Solution trouvé [sur ce post](http://stackoverflow.com/questions/19859216/double-checking-gitlab-installation-gitlab-shell-self-check-failed) )

Si toutes les données sont **vertes**, votre configuration est fonctionnelle !

# Visistez l'interface Web

Vous devriez pouvoir ouvrir votre navigateur et pointer vers le FQDN correspondant (ici [http://glpi.local.fr](http://glpi.local.fr) ) et arriver devant la page suivante :

<figure>
    <img src="{{ site.url }}/images/gitlab/gitlab_accueil.png" alt="">
    <figcaption>Gitlab - Première visite</figcaption>
</figure>

Vous devez définir un nouveau mot de passe pour le compte administrateur. Une fois cela fait, Gitlab va vous rediriger vers la page de connexion et vous demander vos identifiants.

* Login : root
* Mot de Passe : _votreMotDePasse_

Voilà, votre serveur Gitlab est prêt à recevoir vos dépôts !

# LDAP et Utilisateurs

## Utilisateurs standards

Si vous souhaitez que vos utilisateurs s'enregistrent tout seuls, vous n'avez rine à configurer car par défaut Gitlab propose à l'accueil de s'enregistrer. Vous pouvez désactiver cela dans la zone _Admin_(la petite clé en haut à droite) => Settings. décochez la case **Sign-up enabled**.

## Utilisateurs LDAP

Si vous avez un serveur LDAP pour votre entreprise, vous pouvez configurer Gitlab pour qu'il prennent en compte votre Active Directory. Ouvrez votre fichier de configuration principal (`sudo -u git -H vi /home/git/gitlab/config/gitlab.yml`) et rajoutez vos informations comme suis :

```conf
ldap:
  enabled: true
  servers:
   main:
    # Le label sera affiché à l'accueil de Gitlab
    label: 'DOMAINE'
    # Mettez l'IP ou le FQDN de votre serveur LDAP
    # Changez le port si nécessaire
    host: 'XXX.XXX.X.XX'
    port: 389
    # La manière dont Gitlab authentifie l'utilisateur
    uid: 'sAMAccountName'
    # Choisissez la méthode : # "tls" ou "ssl" ou "plain"
    method: 'plain'
    # Mettez les identifiants requis pour se connecter au serveur
    bind_dn: 'DOMAINE\user'
    password: 'xxxxxxxx'
    # Si vos serveur n'est pas un Active Directory, mettez false
    active_directory: true
    # Spécifiez la base dans laquelle doit s'effecuter la recherche
    base: 'DC=domaine,DC=fr'
```

Enregistrez et quittez. Vous devez redémarrez le serveur Gitlab pour que cela soit pris en compte :

```bash
sudo service gitlab restart
```

Vous pouvez toujours vous connecter avec votre compte **root** (heureusement...). Si vous allez dans la zone Admin, vous devriez voir une pastille **verte** à côté de LDAP signalant qu'il est bien activé.

Déconnectez-vous ! Vous devriez voir un onglet **DOMAIN** qui correspond à une connexion utilisateur LDAP et un onglet **Standard** poour votre compte **root** par exemple. Tentez une connexion avec un compte LDAP, normalement cela devrait marcher.

## Espace de nom

Vous allez voir que chaque utilisateur aura un espace de nom (souvent le nom de son compte) pour ses dépôts. Il vous est aussi possible de créer des groupes pour mettre des dépôts et ensuite ajouter les utilisateurs à ce groupe. Vous pourrez aussi déterminer quel utilisateur peut pousser sur le dépôt, mettre en place des "pull request", etc... comme un serveur Github.

# Conclusion

Gitlab n'est pas forcément évident à monter, ni à configurer mais si on est patient, il est en fait très facile d'utilisation. L'équipe fait toujours des documentations très complète, même si parfois ils ne pensent pas à tout évidemment. 

Pour le moment votre serveur est vide, il ne vous reste plus qu'à rajouter des projets et à utiliser Git avec. Je ne ferais pas de tutoriels sur la manière dont on créé un projet car Gitlab donne toutes les commandes nécessaires à la création d'un dépôt, son clonage, le rajout des branches, la création de README, etc...

