---
title: "samba & fstab"
description: "Accès un serveur de fichier Ubuntu 12.10 depuis un poste Windows"
date: "2012-11-22"
tags: ["linux","ubuntu","samba","smb","fstab","xfs"]
---

### Installation

```shell
$ sudo apt-get install samba
```

### Ma configuration perso pour samba

Dans ce qui suit, vous pouvez adapter les paramètres à votre convenance. Perso j'utilise cela 
car j'ai de très bonnes perfs en lecture/écriture des fichiers à travers mon réseau LAN
(environ 8-9 Mo/sec)

On configure le fichier ***/etc/samba/smb.conf***:

```shell
security = user
map to guest = Bad User
socket options=SO_RCVBUF=131072 SO_SNDBUF=131072 TCP_NODELAY
min receivefile size = 16384
use sendfile = true
aio read size = 16384
aio write size = 16384
workgroup = WORKGROUP
netbios name = HOSTNAME
server string = Samba Server %v

### partition pour le partage de données ###
[Storage]
path = /media/Storage/
available = yes
read only = no
browsable = yes
public = yes
writable = yes

```

Les options ci-dessus ont été mises de telles sorte à ce qu'il n'y ait pas de limite pour le transfert de fichier via le
réseau. Pour customiser encore plus cette configuration, voir [là](https://wiki.amahi.org/index.php/Make_Samba_Go_Faster)

Ensuite, si l'on considère que le répertoire de partage est ```/media/Storage```, changer les droits d'accès au répertoire
pour n'importe qui puisse y accéder (lire/écrire).


```shell
$ sudo chown nobody.nogroup /media/Storage/
```

Enfin, redémarrage du service smbd et nmbd (respectivement daemon samba et netbios)

```shell
$ sudo restart smbd 
$ sudo restart nmbd
```

### fstab pour monter la partition samba au démarrage du serveur

Le fichier à modifier : ***/etc/fstab***

Il faut d'abord connaître l'[UUID](http://fr.wikipedia.org/wiki/Universal_Unique_Identifier) de la partition à monter au démarrage.
La commande suivante permet de le savoir:

```shell
$ sudo blkid
```

Enfin dans le fichier fstab, écrire l'entrée suivante pour activer le montage de la
partition souhaitée (on considère que la partition a été formatée avec le système de fichier **[xfs](http://en.wikipedia.org/wiki/XFS)**)

```shell
$ sudo vim /etc/fstab UUID=xxxxxxxxxxx /mnt/Windows xfs users,defaults 0 0
```