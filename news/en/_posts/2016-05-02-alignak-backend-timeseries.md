---
layout: post
title: Timeseries in alignak-backend
lang: en
ref: alignak-timeseries
modified:
description: "Timeseries in alignak-backend"
tags: [alignak]
image:
  feature:
  credit:
  creditlink:
author: ddurieux
comments: true
share:
date: 2016-05-02T20:41:00+01:00
---

In the version 0.4.0 of alignak-backend (currently in development), I have introduce code to add perfdata in timeseries databases.

## What databases

The 2 timeseries databases are:

* graphite (with carbon)
* influxdb (0.9 and above)

## How it works

This is the global schema of alignak-backend works:

<figure>
    <img src="{{ site.url }}/images/alignak-backend/alignak_backend_timeseries.png" alt="">
    <figcaption>Alignak-Backend</figcaption>
</figure>


The broker module alignak-backend send the _check result_ to the alignak-backend.

The backend:

1. store in the alignak-backend database the log (state, check time, output, perfdata, host...)
2. send in graphite and / or influxdb the perfdata


## How to configure timeseries database connection in the alignak-backend

It's simple, in config file you have:

```
; Graphite / carbon for metrics, set GRAPHITE_HOST empty to disable it
; GRAPHITE_HOST']=
; GRAPHITE_PORT']=2004

; Influxdb for metrics, set INFLUXDB_HOST empty to disable it
; INFLUXDB_HOST']=
; INFLUXDB_PORT']=8086
; INFLUXDB_LOGIN']=root
; INFLUXDB_PASSWORD']=root
; INFLUXDB_DATABASE']=alignak
```

Like you see, you can have no timeseries database, graphite, influx or both ;)

## It's cool, but if graphite or influx not available in couple of minutes (restart, update...)

No problem for that, if configured timeseries but not accessible, the alignak-backend store the perfdata internally in the backend (so in 
mongo). It's named _timeseriesretention_.

In configuration file, uncomment the line

```
SCHEDULER_ACTIVE=1
```

Now all 60 seconds, an internal scheduler job check if perfdata are in retention.
If yes, it check the connection with timeserie database. If works, send perfdata, otherwise it will try next time.


