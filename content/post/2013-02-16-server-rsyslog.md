---
title: "Configuration rapide entre un routeur cisco et un serveur rsyslog"
date: "2013-02-16"
tags: ["linux","debian","cisco","rsyslog"]
---

## Contexte
Le serveur rsyslog possède pour adresse IP 172.16.1.253 (sous-réseau en /24)
Le client (routeur cisco) est dans le même sous-réseau (pour faire simple) et a pour adresse IP 172.16.0.254
Le port utilisé par rsyslog et le port UDP 514

## Serveur rsyslog

Il existe un fichier de configuration (`/etc/rsyslogd.conf`) sur le serveur rsyslog permettant d'autoriser les clients à envoyer leur propre logs sur le serveur (ACL)

 
Donc, à la fin de ce fichier, ajouter les lignes suivantes:

```
# Logging for Cisco router 172.25.0.254 
# local7.* /var/log/cisco
$ModLoad imudp
$UDPServerRun 514$
AllowedSender UDP, 172.25.0.254
```

On créé le fichier log :

```
# touch /var/log/cisco
```

Puis redémarrage de rsyslog daemon:

```
# /etc/init.d/rsyslog restart
```

## Routeur Cisco

```
R1#conf t
R1(config)#logging host 172.16.0.253 sequence-num-session
R1(config)#logging trap 7
```

Et voilà! 

Un exemple de sortie standard du serveur rsyslog:

```
tien@localhost:~$ tail -f /var/log/cisco 
Feb 16 23:13:29 172.16.0.254 35: [syslog@9 s_sn="1"]:
*Feb 16 23:13:20.623: %SYS-5-CONFIG_I: Configured from console by console Jun 10 23:13:30 172.16.0.254
Feb 16 23:13:29 172.16.0.254 36: [syslog@9 s_sn="2"]:
*Feb 16 23:13:21.631: %SYS-6LOGGINGHOST_STARTSTOP: Logging to host 172.16.0.253 port 514 started - CLI initiated
```

