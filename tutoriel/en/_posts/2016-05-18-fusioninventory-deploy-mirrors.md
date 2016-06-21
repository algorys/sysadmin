---
layout: post
title: FusionInventory and use mirrors for software deployment
lang: en
ref: fusioninventory-mirrors
modified:
description: How to use mirrors for software deployment with FusionInventory
tags: [tutoriel, fusioninventory, glpi]
image:
  feature:
  credit:
  creditlink:
author: ddurieux
comments: true
share:
date: 2016-05-18T19:08:05+01:00
---

# Deploy with mirrors

You can deploy with p2p, but if you can't / don't want to use this on multi-sites, you can use mirrors.

## What is mirrors?

A mirror is a HTTP server where an agent tries to get the file(s) required to deploy a software.

## How does it work?

### In GLPI

In GLPI, add a mirror in menu *Plugins* > *FusionInventory* > *Deploy* > *Mirror servers*

In the field *Mirror server address*, add the url of your HTTP server for the remote site (think to create the mirror server in the right entity).

For example:

```
http://192.168.43.1/
```

### The mirrors

Install your favorite HTTP server (apache, nginx...) in the remote site.

Copy / synchronize the folder (and sub-folders) *glpi/files/_plugins/fusioninventory/files/repository* in the HTTP server (to the server root in our example).


### The Agent

The agent will receive a list of mirror servers from GLPI.

It will try to use a (some if many servers are defined) mirror server to fetch the file(s) required to deploy the software. If it does not find the file on a mirror server, it gets it utlimately from GLPI.
