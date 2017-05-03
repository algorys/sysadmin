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
author:
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

It is also indicated that the version managers for Ruby are also not supported / recommended and may pose different problems for your future `push` and` pull` on the server.

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
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/
sudo chmod -R u+rwX builds/
sudo chmod -R u+rwX shared/artifacts/
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

