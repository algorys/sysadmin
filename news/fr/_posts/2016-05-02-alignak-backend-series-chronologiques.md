---
layout: post
title: Timeseries dans alignak-backend
lang: fr
ref: alignak
modified:
description: "Timeseries dans alignak-backend"
tags: [alignak]
image:
  feature:
  credit:
  creditlink:
comments: true
share:
date: 2016-05-02T20:41:00+01:00
---

Dans la version 0.4.0 de alignak-backend (actuellement en développement), j'ai introduit du code afin d'ajouter les perfdata dans des bases de données timeseries.

## Quelles bases de données

Les 2 bases de données timeseries sont :

* graphite (avec carbon)
* influxdb (0.9 et au-dessus)

## Comment ça fonctionne

Voici le schéma global de alignak-backend:

<figure>
    <img src="{{ site.url }}/images/alignak-backend/alignak_backend_timeseries.png" alt="">
    <figcaption>Alignak-Backend</figcaption>
</figure>


Le module alignak-backend du broker envoi les _résultats de check_ à alignak-backend.

Le backend :

1. stock dans la base de données de alignak-backend les logs (état, timestamp du check, output, perfdata, hôte...)
2. envoi dans graphite et / ou influxdb le perfdata


## Comment configurer les connexions aux bases de données timeseries dans alignak-backend

C'est simple, dans le fichier de configuration il y a :

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

Comme vous pouvez le constater, il peut ne pas y avoir de bases de données timeseries, graphite, influx ou les deux ;)

## C'est cool, mais si graphite ou influx n'est pas disponible pendant plusieurs minutes (redémarrage, mise à jour...)

Pas de problème pour ça, si les timeseries sont configurées mais ne sont pas accessibles, alignak-backend stock les perfdata en interne dans le backend (donc 
dans mongo). C'est nommé _timeseriesretention_.

Dans le fichier de configuration, décommenter la ligne

```
SCHEDULER_ACTIVE=1
```

Désormais, toutes les 60 secondes, un scheduler interne vérifie si des perfdata sont dans la rétention.
Si oui, il vérifie la connexion à la base de données timeseries. Si ça fonctionne, il envoi les perfdata, sinonil réessayera la prochaine fois.

