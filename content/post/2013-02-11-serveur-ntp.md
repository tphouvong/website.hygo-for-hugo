---
title: "Serveur NTP sous Debian 6.0"
description: "Hombre, No Te Preocupes !"
date: "2013-02-03"
tags: ["linux","squeeze","cisco","ntp"]
---


Network Time Protocol (aka [NTP](http://fr.wikipedia.org/wiki/Network_Time_Protocol)) est un protocole utilisé pour la synchronisation d'horloge des systèmes qui en ont besoin, notamment les matériels réseaux. Les matériels Cisco (mais aussi tout poste possédant un système de journalisation) génèrent des logs permettant de suivre les différent changements qui se sont opérés: un lien physique qui tombe, une session BGP qui monte, MAC Flapping, etc. Et il est général bon de savoir à quel moment exact ce sont déroulés ces changements. NTP est là pour ça. 

NTP utilise le __port UDP 123__ pour la couche transport, et est conçu de manière à résister à la gigue. 

Le serveur utilisé est une __Debian Squeeze 6.0__.

On installe donc du serveur NTP avec les packages suivant:

```shell
# aptitude install ntp ntpdate ntp-server
```

Par défaut, ce serveur NTP tout fraichement installé se synchronise avec le serveur distant __X__.debian.pool.ntp.org, avec X allant de 0 à 4 généralement. Vous pouvez modifier l'url de synchronisation dans le fichier `/etc/ntp.conf` et mettre à la place, par exemple :

```
server 0.fr.pool.ntp.org
server 1.fr.pool.ntp.org
server 2.fr.pool.ntp.org
server 3.fr.pool.ntp.org
```

Une fois que c'est fait, vous pouvez vérifiez que le port 123 est ouvert sur le serveur Debian :

```shell
# netstat -atune|grep 123
```

ou encore

```
# lsof -ni|grep ntp
```

Tant que votre système affiche que le port 123 est en écoute (LISTEN) pour n'importe quelle adresse
(i.e. pas de Pare-feu bloquant le port 123, et que netstat affiche `0.0.0.0:123` (resp. lsof avec UDP `*:ntp`),
tout est bon, et vous pouvez configurer vos matériels réseaux pour les synchroniser avec le serveur
NTP de Debian.
