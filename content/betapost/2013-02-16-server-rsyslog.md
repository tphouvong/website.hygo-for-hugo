---
title: "Configuration rapide entre un routeur cisco et un serveur rsyslog"
date: "2013-03-16"
tags: ["linux","debian","cisco","rsyslog"]
---

## Serveur rsyslog
Rsyslog
Rsyslog sur 172.25.0.1:514 (UDP), adresse IP du client syslog à autoriser (par ex : 172.25.0.254)
/etc/rsyslogd.conf à la fin du fichier, rajouter:
# # Logging for Cisco router 172.25.0.254 # local7.* /var/log/cisco
#
provides
UDP
$ModLoad
$UDPServerRun 514$AllowedSender UDP, 172.25.0.254

Puis création du fichier log, taper dans le terminal :

syslog

reception
imudp

# touch /var/log/cisco

Puis redémarrage de rsyslog daemon:
# /etc/init.d/rsyslog restart

Cisco Router
R1#conf tR1(config)#logging host 172.25.0.1 sequence-num-session
R1(config)#logging trap 7

et voilà! un exemple de sortie standard du serveur rsyslog:
tien@localhost:~$ tail -f /var/log/cisco Jun 10 23:13:29 172.25.0.254 35: [syslog@9 s_sn="1"]:
*Jun 11 01:13:20.623: %SYS-5-CONFIG_I: Configured from console by console Jun 10 23:13:30
172.25.0.254
36:
[syslog@9
s_sn="2"]:
*Jun
11
01:13:21.631:
%SYS-6LOGGINGHOST_STARTSTOP: Logging to host 172.25.0.1 port 514 started - CLI initiated
Posted in:Cisco,Linux,Réseaux | Tagged:Cisco,Debian,Linux,Rsyslog | With 0 comments

