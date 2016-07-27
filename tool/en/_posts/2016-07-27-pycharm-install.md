---
layout: post
title: Install PyCharm on Linux
lang: en
ref: pycharm
modified:
description: How to install Pycharm on Linux
tags: [tutoriel, python]
image:
  feature:
  credit:
  creditlink:
author: algorys
comments: true
share:
date: 2016-07-27T09:00:05+01:00
---

# Introduction

For those who want to contribute to projects or to develop with python, PyCharm is a good IDE. He provides many features like many language (Python, Markdown, Yaml...) and great tools to develop your python library or software.

# Prerequisites

You just need a python version from 2.4 up to version 3.5. Currently, your package provides it:

```bash
# Example on Debian for python 2.7
sudo apt-get install python2.7
```

You need to have access to a **root** user.

# Download Archive

First you need to download PyCharm archive [here](https://www.jetbrains.com/pycharm/download/#section=linux). If you have non licence, download **Community Edition**.

This will download a `tar.gz` archive. Once finished, uncompress it:

```bash
tar xzvf pycharm-community-2016.2.tar.gz -C /tmp/
```

# Remove previous install

If you have already install PyCharm on your PC, remove old folder and symlinks:

```bash
sudo rm -R  /opt/pycharm-community
sudo rm /usr/local/bin/pycharm
sudo rm /usr/local/bin/inspect
```

# Install PyCharm

You now have a folder called **pycharm-community-2016.2** in `/tmp`. Let's move to `/opt` folder:

```bash
sudo mv /tmp/pycharm-community* /opt/pycharm-community
```

# Making PyCharm Symlinks

Now create symlinks for your user:

```bash
sudo su -c "ln -s /opt/pycharm-community/bin/pycharm.sh /usr/local/bin/pycharm"
sudo su -c "ln -s /opt/pycharm-community/bin/inspect.sh /usr/local/bin/inspect"
```

# Launch PyCharm

Now you that's done ! You've just to launch PyCharm:

```bash
pycharm
```

If you had previous install, PyCharm attempt to import your previous settings ("I want to import my settings from a previous version (~/.PyCharm2016.1/config)". Just click on **OK**.

PyCharm is now launch, and he'll update your plugins. Restart it after this process.

# Conclusion

Install of PyCharm is very easy and if you develop in Python, that's one of the best IDE you can find on Linux.



