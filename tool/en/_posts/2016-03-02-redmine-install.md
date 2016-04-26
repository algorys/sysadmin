---
layout: post
title: Install Redmine 3.0
lang: en
ref: redmine
modified:
description: How to install Redmine
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

In business, there are always several projects that require monitoring, management of tickets, documents, time spent, etc. Some paid software do very well this work but still a cost. To overcome this, there are open source project managers, whose [Redmine](http://www.redmine.org/) belongs.

It has the advantage of discussing with many other servers (such as Gitlab or Jenkins) and has a lot of features. Many plugins are also available and you can create yourself.

# Tutorial Prerequisites

We'll see how to install Redmine in its latest stable version (currently 3.2). Installing the base server to a production server, with its Web server and service.

For that we will have make sure you have the following prerequisites :

* An Ubuntu machine (or Debian-like) with **root** access
* The curl program installed (`sudo apt-get install curl`)
* Knowledge of Linux and patience ...

# Redmine Prerequisites

Redmine needs a lot of things to work. It uses Ruby and MySQL (or other database), and the Gem programs, Bundler and several other outbuildings. It is always important to consult [prerequisite Redmine](http://www.redmine.org/projects/redmine/wiki/RedmineInstall#Requirements) before embarking on an installation.

To date (March 2016) , this is what the documentation says :

| Version de Redmine | Versions de Ruby supportées    | Version de Rails utilisée |
|:------------------:|:------------------------------:|:-------------------------:|
| Version en cours   | ruby 1.9.3, 2.0.0, 2.1, 2.2 | Rails 4.2                 |
| Redmine 3.0        | ruby 1.9.3, 2.0.0, 2.1, 2.2 | Rails 4.2                 |
| Redmine 2.6        | ruby 1.8.7, 1.9.2, 1.9.3, <br>2.0.0, 2.1, 2.2, jruby-1.7.6 | Rails 3.2 |


Supported Database : MySQL 5.0 or PostgreSQL 8.2 or higher, Microsoft SQL Server 2008 (Redmine 2.X) or 2012 (Redmine 3.X) and _SQLite 3_.

As you can see, Redmine supports quite a different version of Ruby and SQL database. What is practical without being. Ruby and Rails are sensitive enough when we blend multiple versions or their libraries, some versions of Ruby have known bugs...

In this tutorial, we will use Ruby 2.1 and MySQL to install Redmine 3.X.

## Create a dedicated user

For reasons of good practice, it is always better to create a user who will take care of the created service. Personally, I created a user who is named `redmine` to keep a logical but it's not an obligation.

I also chose to put his _HOME_ in a different directory from the usual _HOME_ (`/ home /`) but this is not compulsory too.

```bash
sudo mkdir /usr/local/home
sudo useradd -s /bin/bash -m -d /usr/local/home -g root redmine
```

And connect with your newly created user :

```bash
sudo su - redmine
```

In the remainder of this tutorial you normally keep this user.

## Install Ruby

To install Ruby, we could go through packages provided by your distribution. But it is not necessarily up to date and offer perhaps not recent enough version to install Redmine 3.X. I suggest you go through [RVM](https://rvm.io/).

The advantage with RVM is that it allows you to easily manage your versions of Ruby and especially to be able to update them in no time.

To install RVM, type the following command (as user **redmine** !) :

```
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
```

> **Note :** It is possible that getting RVM differs on Debian or other distribution. For Ubuntu I had to force port 80 and point to `keyserver.ubuntu`. Official documentation recommends point to the key server : `hkp://keys.gnupg.net`.

Normally if all goes well , you should have the following output :

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

That's done, rvm is installed. As said in the installation, to start using it you have to type the following command :


```bash
source /usr/local/home/redmine/.rvm/scripts/rvm
```

Now that you have rvm installed, you can see the different versions of Ruby available by typing `rvm list known`. It only remains to indicate what version of Ruby you want to install with the command :

```bash
rvm install 2.1.5
```

The program will ask the `root` password to continue installation and install all alone required dependencies (gcc, g++, make, zlib, etc.). If all goes well, you should have Ruby installed. You can check by typing `ruby --version`.

## Install MySQL

To install MySQL, it's much easier. Simple installation by deposits will be sufficient, and a library in addition to the adapter `mysql2`.

```bash
sudo apt-get update
sudo apt-get install mysql-server libmysqlclient-dev
```

You may need to create a password for the MySQL `root` account. Remember the well in order to access it later.

Congratulations ! You have all the prerequisites to install Redmine.

## Download the archive

To do this, go to the last page of [stable releases of Redmine](http://www.redmine.org/projects/redmine/wiki/Download#Stable-releases) and download the archive you want. Unzip it in the _HOME_ you have chosen and rename `redmine-X.X.X` to `redmine`.

In our example, this gives :

```bash
cd ~; wget http://www.redmine.org/releases/redmine-3.2.0.tar.gz
tar -xzvf redmine-3.2.0.tar.gz
mv redmine-3.2.0 redmine
```

## Configure SQL Database

So that Redmine can store and organize all the information, it needs a database.

### Configure SQL

Log in with your SQL `root` account and type the following lines (replacing `your_password` by a secure password) :

```bash
mysql -u <utilisateur_root> -p
```

```sql
mysql> CREATE DATABASE redmine CHARACTER SET utf8;
mysql> CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'your_password';
mysql> GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```

You can then leave MySQL by typing `Ctrl-D` or `exit`.

> **Note :** again the SQL user and the base are called _redmine_ but this is not mandatory. Feel free to call them as you like, as long as you will apply the changes accordingly in the configuration files.

### Configure SQL for Redmine

Redmine works with configuration files with the extension `.yml`. And its creators are nice, they put examples everywhere. What interests us now is say to Redmine on which Database it should work, so we will copy the `conf/database.yml.example` :

```bash
cd ~/redmine
cp config/database.yml.example config/database.yml
```

Open the copied file (vi `config/database.yml`) and edit it to suit your configuration. In our example, it should look like this :

```conf
# Examples for PostgreSQL, SQLite3 and SQL Server can be found at the end of the file.
# Indent lines have to be 2 spaces (not tabulation).

production:
  adapter: mysql2           # adapter use
  database: redmine         # name of database
  host: localhost           # hostname
  username: redmine         # SQL user
  password: "your_password" # user password
  encoding: utf8            # database encoding
```

Personally, I deleted other configurations that were not useful to me, and I can find them on the sample file.

## Install dependencies

Now Redmine needs Bundler to operate and manage the dependencies of its Gems. To install Bundler :

```bash
cd ~/redmine
gem install bundler
```

Then, with Bundler you will be able to install Redmine Gems :

```bash
bundle install --without development test rmagick
```

If all goes well, you should see several installation lines (of different gems) and at the end the following lines :

```bash
Bundle complete! XX Gemfile dependencies, XX gems now installed.
Gems in the groups development, test and rmagick were not installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

> **Note :** as you could see, we tell Bundler not install gems to RMagick. This gem is optional and used to use ImageMagick to manipulate PDF and PNG to export data. If you still want to install it, make sure you have the required dependencies before you run `bundle install`. For Ubuntu, you need to install the packages `imagemagick` and `libmagickwand-dev` beforehand.

## Generate an encryption key

This step will enable Rails to have a key to encrypt cookies from your sessions. To generate this key type the following command :

```bash
bundle exec rake generate_secret_token
```

## Creating the Schema Base

Redmine will have then need to structure its database to work. It should match your environment, the environment here is _production_.

```
RAILS_ENV=production bundle exec rake db:migrate
```

You will have a lot of running lines :

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

You must then insert the default data (and the language ) in your base :

```bash
RAILS_ENV=production REDMINE_LANG=fr bundle exec rake redmine:load_default_data
```

This should display the following output :

```bash
Default configuration data loaded.
```

Your base is updated and contains all that is necessary for Redmine operational.

## Permissions Files

You will then need to ensure that certain files are accessible and give them the following permissions :

```bash
mkdir -p tmp tmp/pdf public/plugin_assets
chmod -R 755 files log tmp public/plugin_assets
```

If you have not created your redmine _HOME_ folder in the user or you made a different way, you must ensure that user have rights to the following folders: `files` , `log`, `tmp` and `public/plugin_assets`.

Congratulations ! You finally have a redmine's server that is ready !

# Test Installation

To test your Redmine server, you will now execute the following command :

```bash
bundle exec rails server webrick -e production
```

> **Warning :** This command is only used to test the installation and should not be used in "production" ! We will see later how to mount the Redmine service.

Open your browser and point to the following address : [http://localhost:3000/](http://localhost:3000/). If you have a console, download the index of that address (`wget http://localhost:3000/`).

> **Tip :** if you made this tutorial on a virtual machine that has no GUI (there's a good chance that's the case !), you can ensure that the localhost server becomes yours. Mount an SSH tunnel between your PC and the server, with port forwarding (here the port is _3000_). Here's an example : `ssh -L 3000:127.0.0.1:3000 login@ip_server`. Then point to address _http://localhost:3000_ with your browser and you should arrive on Redmine interface !

To log, just use the following identifiers :

* Login: admin
* Password: admin

I advise you to change it later, of course.

You can stop the server by typing `Ctrl-C`.

# Create Redmine Service

You finally have an operational Redmine server ... But against it is only accessible on the local loop of the server (localhost) and involves going on port 3000. In addition, it shows us that we can't use Redmine this way in production. In short, all this is not very pretty.

We will create a service using the _unicorn_ gem and a Web server with Nginx.

## Install Unicorn

To use _unicorn_, we must first install it. Do not touch the Gemfile Redmine and instead use the `Gemfile.local` file dedicated to this kind of manipulation. Add _unicorn_ gem to that file ( `vi Gemfile.local`) :

```conf
gem 'unicorn'
```

Then re-launch Bundler to install the new gem :

```bash
bundle install --without development test rmagick
```

You should see the installation of new gems (_unicorn_ and its dependencies).

## Configuration of Unicorn

Unicorn will now need a configuration file named `unicorn.rb`. We will place the file in the folder `config` Redmine (`vi config/unicorn.rb`) and add the following lines :

```ruby
worker_processes 4
working_directory "/usr/local/home/redmine/redmine"
listen "/var/run/redmine/redmine-unicorn.sock", :backlog => 64
timeout 30

pid "/var/run/redmine/unicorn.pid"

stderr_path "/var/log/redmine/unicorn.stderr.log"
stdout_path "/var/log/redmine/unicorn.stdout.log"
```

Now create the corresponding folders and security credentials :

```bash
sudo mkdir /var/run/redmine
sudo chown redmine:redmine /var/run/redmine/
sudo mkdir /var/log/redmine/
sudo chown redmine:redmine /var/log/redmine/
touch /var/log/redmine/unicorn.stderr.log
touch /var/log/redmine/unicorn.stdout.log
```

Here it is, _unicorn_ is configured. We will now create the associated service.

## Unicorn-Redmine service

To create a service for Redmine, we have to add it in `/etc/init.d/`. Be careful to check the paths that you will put in the file, a single error in one of the paths and the service will not work.

Fortunately there is always the logs defined in the previous step to help you debug in case of problems !

Create a file ( `sudo vi /etc/init.d/redmine_unicorn`) and add the following lines :

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

If you have carefully followed all the tutorial, you normally not have to make any changes on this file. But still check the different variables defined in it and as said before different path.

Then give the rights on this file and activate the service at server startup :

```bash
sudo chmod 755 /etc/init.d/redmine-unicorn
sudo update-rc.d redmine-unicorn defaults
```

You can now start the redmine-unicorn service :

```bash
sudo service redmine-unicorn start
```

If you have any error, check your logs (`tail -f /var/log/redmine/*`).

# Web Server Installation

It remains only to install and configure a virtual server for our Redmine server. We use Nginx as a Web Server.

## Installing Nginx

To install Nginx :

```bash
sudo apt-get install nginx
```

## Web Server Configuration

Once installed, we will configure the Web server. To avoid conflicts with the default server, you can either delete the `default` file located in `/etc/nginx/sites-available/` or comment lines with ports 80 :

```nginx
server {
    #listen 80 default_server;
    #listen [::]:80 default_server ipv6only=on;
```

Create a configuration file for your virtual server ( `vi /etc/nginx/sites-available/redmine` ) and add the following lines :

```nginx
# Name and define socket
upstream redmine_unicorn {
    server unix:/var/run/redmine/redmine-unicorn.sock fail_timeout=0;
}
# Define server
server {
    listen 80 default deferred;
    client_max_body_size 4g;

    # Name of server and his root folder
    server_name redmine.local.fr;
    root /usr/local/home/redmine/redmine/public;

    keepalive_timeout 5;

    try_files $uri/index.html $uri.html $uri @app;

    # Configuration of proxy
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

Save and quit. Now your virtual host is configured, you have just to restart Nginx :

```bash
sudo service nginx restart
```

Congratulations ! You should be able to access your production server via the following URL : [http://redmine.local.fr](http://redmine.local.fr).

## Some solutions to common errors

If you have only the home page Nginx:

* Make sure you have commented (or deleted) the server `default`.
* Check that your nginx `redmine` configuration file is not wrong (it may lack a `;` at end of line for example).

If you have a blank page or error :

* Check the logs of nginx: `sudo tail -f /var/log/nginx/error.log`

Do not forget to restart **Nginx** each changes on its configuration files.

# Conclusion

You now know how to make a Redmine server. Redmine then has many configurations such as rights, dependencies between projects, various tickets, custom fields, etc. It also has a catalog of plugins well supplied, including one that lets you synchronize your Redmine users via Active Directory. Or a plugin that will allow you to link Redmine tickets to Git commits (or Gitlab server).
