---
layout: post
title: Big rework alignak
lang: en
ref: alignak-bigrework
modified:
description: "Big rework alignak"
tags: [alignak]
image:
  feature:
  credit:
  creditlink:
comments: true
share:
date: 2016-05-10T08:21:50+01:00
---

Sebastien Coavoux and David Durieux have worked hard these last weeks in Alignak and they have made these modifications:

* Use uuid in all objects instead of id
* Unlink almost all items: attribute of object are no instance of other objects except for a few cases (commandcall / check_period and acknowledgemnt)
* Replace pickle by manual serialization: return a json representation of objects.
* Drop of base64 / zlib manual serialisation in HTTP and use CherryPy facilities to have json
* Rework class inheritance: instantiation of object is more generic
* Clean unused functions and class
* Increase CherryPy queue for larger environnment


So you can ask, is it better?

YES it is!

We have a better code readeable, better performances to serialize and send to other daemons (win between 10 to 50% of time, CPU and memory).

You will be able to see the difference in next release of Alignak ;)
