---
layout: post
title: Install Gitlab Community
lang: en
ref: gitlab
modified:
description: How to install Gitlab, a github-like at home
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

[Gitlab](https://about.gitlab.com/) is a server near to [Github](https://github.com). He let to manage and centralize your [Git](https://git-scm.com/) repositories with a Web user Interface. Gitlab also provide an issue manage system, a powerfull API and many other features compared to a simple `bare` repository.

The advantage with such a server is that it allows you to see the status of your repositories from any platform / device and most importantly it brings a complete group management and tools to chat with other servers (Like Jenkins for example).

This tutorial allows you to install Gitlab in its `Community` version. He resume the [official tutorial](http://doc.gitlab.com/ce/install/installation.html#packages-dependencies) already well done.

# Prerequisites

To deploy a Gitlab server, you should have a good knowledge of Git and Linux (and Ruby if possible) to understand to understand the interest and especially its functioning.

* A Linux server with `root` rights.

_This tutorial was performed on an Ubuntu Server 14.04 LTS and 16.04 LTS_

# Dependencies

Like all softwares, Gitlab need some external libraries to work. Ensure your server is up-to-date and install the following:

```bash
sudo apt-get update
sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake 
```

If you want to use [Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)), install also following library:

```bash
sudo apt-get install libkrb5-dev
```

## Install Git

You should have a Git version upper or equal to `2.7.4`. In many distribution, Git will not be up-to-date with the current version of Git.

To get last Git version, add the repository:

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
# Then install "git-core"
sudo apt-get install git-core
```

If you want ot be sure to have a compatible version, run:

```bash
algorys@srv-gitlab:~$ git --version
git version 2.7.4
```

> **Note:** I've installed a Gitlab server with a 1.9.1 version and Gitlab is working well. But, for **safety reasons**, it is better to have the latest version installed.

Now you have all requirement to install Gitlab, but the road is still long...

# Ruby

Gitlab need at least Ruby 2.1.x for Gitlab < 9.0 and Ruby 2.3 for Gitlab >= 9.0. So be sure to install the right version !

It is also indicated that the version managers for Ruby are also not supported / recommended and may pose different problems for your future `push` and `pull` on the server.

## Ruby 2.1

To install Ruby 2.1, first remove your current version if needed:

```bash
sudo apt-get remove rubyX.x
```

Where `X.x` is your Ruby version number.

Then download Ruby:

```bash
mkdir /tmp/ruby && cd /tmp/ruby
curl -O --progress https://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.8.tar.gz
echo 'c7e50159357afd87b13dc5eaf4ac486a70011149  ruby-2.1.8.tar.gz' | shasum -c - && tar xzf ruby-2.1.8.tar.gz
```

And compile and install:

```bash
cd ruby-2.1.8
./configure --disable-install-rdoc
make
sudo make install
```

It may take some time before ending, so you can drink a coffee while waiting.

## Ruby 2.3

On recent "distro", Ruby 2.3 is already installed. If not just follow the same procedure as above by changing the version number.

## Bundle

Once Ruby installed, you've to install the `Bundler` Gem:

```bash
sudo gem install bundler --no-ri --no-rdoc
```

> **Note:** if you have headers problems, just install `rubyX.x-dev` packages to solve it.

Now Ruby is installed and ready to use.

# Go

Since Gitlab 8.0, HTTP requests of git are done with `gitlab-workhorse`. This a daemon like written in Go. To install it, you need Go compiler.

The foolowing commands are for a 64-bit Linux. Adjust archive according to your platform by going on the [Go Download Page](https://golang.org/dl/) !

```bash
curl -O --progress https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
echo '43afe0c5017e502630b1aea4d44b8a7f059bf60d7f29dfd58db454d4e4e0ae53  go1.5.3.linux-amd64.tar.gz' | shasum -a256 -c -
sudo tar -C /usr/local -xzf go1.5.3.linux-amd64.tar.gz
sudo ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
# Delete useless archive
rm go1.5.3.linux-amd64.tar.gz
```

# Node

Since **Gitlab 8.17**, Gitlab use `node >= v4.3.0` to compile javascript assets and `yarn >= v0.17.0`to manage its dependencies.

```bash
# Install node v7.x
curl --location https://deb.nodesource.com/setup_7.x | sudo bash -
sudo apt-get install -y nodejs

# Install yarn
# In Official Documentation, here is the given command:
curl --location https://yarnpkg.com/install.sh | sudo -u git -H bash -
# Personally, Gitlab could not detect "yarn" thereafter
# If this is your case, install from packages: https://yarnpkg.com/en/docs/install
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
```

# Git User

You have to create a dedicated user to Gitlab, without password. By convention, we give as name `git`:

```bash
sudo adduser --disabled-login --gecos 'Gitlab' git
```

Later, you will see that many things will be found in the `$HOME` directory of this user.

# Database

Gitlab does not recommend **MySQL** ! It is however possible to use it by going on [MySQL Documentation](http://doc.gitlab.com/ce/install/database_mysql.html).

Gitlab recommend to use **PostgreSQL >= 9.1** ! So we do as they want:

```bash
sudo apt-get install -y postgresql postgresql-client libpq-dev postgresql-contrib
```

Then define `git` user inside:

```bash
sudo -u postgres psql -d template1 -c "CREATE USER git CREATEDB;"
# Create "pg_trgm" extension
sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
```

Create the Database dedicated to "production":

```bash
sudo -u postgres psql -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"
```

Then try to connect:

```bash
sudo -u git -H psql -d gitlabhq_production
```

You should have the following output:

```sql
psql (9.x.x)
Type "help" for help.

gitlabhq_production=>
```

Check if `pg_trgm` extension is enabled:

```sql
SELECT true AS enabled
FROM pg_available_extensions
WHERE name = 'pg_trgm'
AND installed_version IS NOT NULL;
```

If it's enbled, you should see:

```sql
enabled
---------
 t
(1 row)
```

Type `\q` or press `CTRL-D` to quit PostgreSQL.

# Install Redis Server

Redis is a powerfull Database management system and is required by Gitlab. A minimum `2.8` version is required! On Debian like, this package is normally up-to-date:

```bash
sudo apt-get install redis-server
```

> If you have not the required version, you have to add the repository by following [Gitlab Redis Documentation](http://doc.gitlab.com/ce/install/redis.html).

Now, you've to give `redis` rights to `git` user by adding him in group:

```bash
sudo usermod -aG redis git
```

And modify the configuration file like this (`sudo vi /etc/redis/redis.conf`):

```conf
# Change port to 0
port 0
# Set socket path and permissions for redis
unixsocket /var/run/redis/redis.sock
unixsocketperm 770
```

Then restart **redis-server**:

```bash
sudo service redis-server restart
```

Ready is Redis ;)

# Gitlab

Finally ! We'll take care of Gitlab!

Go to the `git` home folder and clone the repository:

```bash
cd /home/git
# Change the Gitlab version to the one you want. Here is the 9.1:
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 9-1-stable gitlab
```

Now, we have to configure Gitlab according to what we installed before:

```
cd /home/git/gitlab
sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
```

Open the `gitlab.yml` file (`sudo -u git -H vi config/gitlab.yml`). You have at the top of the file, the main changes you have to do. But here is a resume:

```yaml
# Buy default, "host" is at `localhost`. You can put your FQDN here instead.
host: localhost
# Chosse an email for emails from this server
email_from: example@example.com
# Check your Git path
git:
  bin_path: /usr/bin/git
```

Of course, you'll have other settings to configure after, like LDAP accounts (see at the end of this tutorial).

Once done, copy the following file:

```bash
sudo -u git -H cp config/secrets.yml.example config/secrets.yml
sudo -u git -H chmod 0600 config/secrets.yml
```

Ensure Gitlab can write in the following folders:

```bash
sudo chown -R git log/
sudo chown -R git tmp/
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/
```

Then, create a folder `public/uploads/` and ensure that nobody can write in:

```bash
sudo -u git -H mkdir public/uploads/
sudo chmod 0700 public/uploads
```

And finally, change the permissions for Gitlab-CI and Gitlab Pages:

```bash
sudo chmod -R u+rwX builds/
sudo chmod -R u+rwX shared/artifacts/
sudo chmod -R ug+rwX shared/pages/
```

Now copy the `unicorn.rb` file and edit it:

```bash
sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
sudo -u git -H vi config/unicorn.rb
```

You have to define the worker processes number in this file. For a server with 2GB of RAM, the number is 3 maximum. You can see number of processor of your server by typing the followin command: `nproc`. To define the worker number, count processors number + 1.

```yml
worker_processes 3
```

Save and quit this file.

Then, you have to copy the `rack_attack.rb` configuration file:

```bash
sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
```

And you have to dconfigure git for Gitlab. This settings will be used when you edit something on Web interface and others:

```bash
sudo -u git -H git config --global core.autocrlf input
sudo -u git -H git config --global gc.auto 0
sudo -u git -H git config --global repack.writeBitmaps true
```

Finally, you need to copy the Redis configuration file and edit it if you have modify the socket path before:

```bash
sudo -u git -H cp config/resque.yml.example config/resque.yml
sudo -u git -H vi config/resque.yml
```

```conf
# Changement à faire pour le path du socket
# production: unix:/var/run/redis/redis.sock
production: unix:/path/to/new/sock.sock
```

## Configure DataBase for Gitlab

Now you've to copy and check the `database.yml` file:

```bash
sudo -u git cp config/database.yml.postgresql config/database.yml
sudo -u git -H vi config/database.yml
# Check the file according to previous configurations
```

And make this file readable to user git only:

```bash
sudo -u git -H chmod o-rwx config/database.yml
```

## Install Gems

Ruby needs some Gems to work (like module in Python or other languages). You must have a bundle versigreater than or equal to `1.5.2` (run `bundle -v` to see it). If this is the case, run gems installation:

```bash
# For an installation with PostgreSQL.
# If you have choosen MySQL, replace "mysql" by "postgres"
sudo -u git -H bundle install --deployment --without development test mysql aws kerberos
```

You should see the following output:

```bash
Fetching gem metadata from https://rubygems.org/...
Installing ...
[...]
Bundle complete! 173 Gemfile dependencies, 264 gems now installed.
Gems in the groups development, test, mysql, aws and kerberos were not installed.
Bundled gems are installed into ./vendor/bundle.
```

> **Note:** it's possible, depending on your connection, that some **gems** take a long time to be installed. Be patient and do not interrupt the installation. If there is any error, `bundle` is usually verbose enough to tell you what to do.

> **Note:** if you have the following error: `can't find header files for ruby at /usr/lib/ruby/include/ruby.h` is that you are missing the ruby headers files. In this case, just run: `sudo apt-get install ruby2.x-dev` to install them.

# Install Gitlab Shell

The _Gitlab Shell_ is a repository management software especially developed for Gitlab. To install it, run the `gitlab-shell` task:

```bash
# Change Redis URL if needed !
sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production SKIP_STORAGE_VALIDATION=true
```

This will clone the corresponding repository in `/home/git/gitlab-shell` and create `repositories` in git _HOME_ folder. Here is the output:

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

Check after the configuration file (`sudo -u git -H vi /home/git/gitlab-shell/config.yml`). It must match your installation, especially if you intend to use a DNS and an FQDN:

```conf
gitlab_url: http://gitlab.local.fr/
```

Now we can install `gitlab-workhorse`.

# Install Gitlab-Workhorse

Go on the _HOME_ git folder and clone repository:

```bash
cd /home/git
# For Gitlab 8.x
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-workhorse.git
cd gitlab-workhorse
sudo -u git -H git checkout v0.7.1
sudo -u git -H make
# For Gitlab 9.x
sudo -u git -H bundle exec rake "gitlab:workhorse:install[/home/git/gitlab-workhorse]" RAILS_ENV=production
```

# Init Data Base

Now we can initialize the database. If you encounter the following error:

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

To solve it, install the following:

```bash
# Adaptez à la version de votre PostgreSQL
sudo apt-get install -y postgresql-contrib-9.3
```

Initialize database:

```bash
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
# Answer "yes" to create tables
```

At the end, you should have the following output:

```bash
== Seed from /home/git/gitlab/db/fixtures/production/001_admin.rb
Administrator account created:

login:    root
password: You'll be prompted to create one on your first visit.
```

# Secure your secrets.yml file

This file keep stores encryption keys for sessions and secure variables. Backup secrets.yml someplace safe, but don't store it in the same place as your database backups. Otherwise your secrets are exposed if one of your backups is compromised.

# Install Init Script

Now you need init script to start, stop and restart your Gitlab server properly:

```bash
# If you do not use the default folders, adapt this file according to your configuration
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
sudo update-rc.d gitlab defaults 21
```

Enable rotation logs:

```bash
sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
```

# Gitaly (for Gitlab 9.1 only)

Since **Gitlab 9.1**, Gitaly is an optional component. It is OK to wait with setting up Gitaly until you upgrade to GitLab 9.2 or later.

```bash
sudo -u git -H bundle exec rake "gitlab:gitaly:install[/home/git/gitaly]" RAILS_ENV=production
```

Then configure it:

```bash
sudo chmod 0700 /home/git/gitlab/tmp/sockets/private
sudo chown git /home/git/gitlab/tmp/sockets/private
cd /home/git/gitaly
sudo -u git -H cp config.toml.example config.toml
# Then configure it if needed
sudo -u git -H vi config.toml
```

Enable it in **init** file of Gitlab (`sudo vi /etc/init.d/gitlab`):

```conf
gitaly_enabled=true
```

```bash
# Update daemons
sudo systemctl daemon-reload
```

Then in Gitlab configuration file (`sudo -u git -H vi /home/git/gitlab/config/gitlab.yml`):

```conf
gitaly:
    enabled: true
```

# Check Installation

Now that most files are configured and ready, it must be verified that all this can work. To do this, you will need to run the following test:

```bash
# Check application status
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
# Command output (may differ depending on the installation):
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

# Compile "assets"

Now you've to compile the Gitlab assets:

```bash
# If you use "yarn"
sudo -u git -H /home/git/.yarn/bin/yarn install --production --pure-lockfile
# Then
sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production NODE_ENV
```

This may take some time, so be patient...

# Start Gitlab

If all previous commands are passed, you can start your Gitlab server !

```bash
sudo service gitlab start
# Output or "status" command output
Starting GitLab Unicorn
Starting GitLab Sidekiq
Starting gitlab-workhorse

The GitLab Unicorn web server with pid 24274 is running.
The GitLab Sidekiq job dispatcher with pid 24320 is running.
The gitlab-workhorse with pid 24302 is running.
GitLab and all its components are up and running.
```

# Web Inerface with Nginx

Now that your server is running, it leave just the Web Interface.

Gitlab recommends Nginx:

```bash
sudo apt-get install -y nginx
```

## Configure Site

As from the beginning, you already have a sample file to copy and you simply activate the site with a symbolic link:

```bash
sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab
sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab
# Delete the "default" server to avoid conflicts
sudo rm -f /etc/nginx/sites-enabled/default
```

Configure file to suit your configuration (`sudo vi /etc/nginx/sites-enabled/gitlab`):

```conf
# Add your FQDN
server_name gitlab.local.fr;
```

## Test Nginx and Restart

Confirm your Nginx configuration with the following command:

```bash
sudo nginx -t
```

Then restart Nginx:

```bash
sudo service nginx restart
```

That's done !

# Visit Web Interface

You should be able to open your browser on [http://gitlab.local.fr](http://gitlab.local.fr) as setting in Nginx file. The following page should be displayed:

<figure>
    <img src="{{ site.url }}/images/gitlab/gitlab_accueil.png" alt="">
    <figcaption>Gitlab - First Visit</figcaption>
</figure>

You should define a new password for admin account ! Once done, Gitlab will redirect you on login page.

* Login: root
* Password: _thePasswordYouHaveEnterBefore_

Congratulations, your Gitlab server is ready to receive your first repository !

# LDAP and Users

## Standard users

If you want that your users register themselves, you have nothing to do. By default, Gitlab let users register on Web Interface. You ca deactivate this feature in _Admin_ area (the key icon on top right) and under _Settings_. Uncheck **Sign-up enabled**.

## LDAP Users

If you have a LDAP server, you can configure it in Gitlab. Open the Gitlab configuration file (`sudo -u git -H vi /home/git/gitlab/config/gitlab.yml`) and add your LDAP informations:

```conf
ldap:
  enabled: true
  servers:
   main:
    # Label who be displayed at Gitlab start page
    label: 'DOMAIN'
    # Set LDAP IP or FQDN
    # Change port if needed
    host: 'XXX.XXX.X.XX'
    port: 389
    # How Gitlab authenticates users
    uid: 'sAMAccountName'
    # Choose method: # "tls" or "ssl" or "plain"
    method: 'plain'
    # Put account data of a user who had sufficient rights
    bind_dn: 'DOMAIN\user'
    password: 'xxxxxxxx'
    # If your server is not Active Directory server
    active_directory: true
    # Specify research base
    base: 'DC=domaine,DC=fr'
```

Save and quit. You should restart Gitlab to take in count:

```bash
sudo service gitlab restart
```

You will always be able to log in with your ** root ** account (fortunately ...). If you go in Admlin area, you should see a "green" chip near LDAP. You should also see a new Tab with your Domain name (the one you've set for "label" before).

## Namespaces

You will see that each user will have a namespace (often the name of his account) for his repositories. You can also create groups to put repository and then add users to this group. You can also determine which user can push on the repository, pull, make pull requests... as a Github server.

# Conclusion

Gitlab is not necessarily obvious to create, nor to configure but if you're patient, it is in fact very easy to use. The Gitlab team always makes documentation very comprehensive !

For the moment your server is empty, you just have to add projects and use Git with. I would not do any tutorials on how to create a project because Gitlab gives all the necessary commands to create a repository, cloning, adding branches, creating README...
