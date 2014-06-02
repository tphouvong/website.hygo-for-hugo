---
title: "Installer XBMC sur Ubuntu 12.10"
description: "Un Media center maison"
date: "2012-11-23"
tags:
  - "linux"
  - "xbmc"
  - "htpc"
  - "ubuntu"
---

## Xbox Media Center...

<img class="img-center center-block" src="/img/xbmc.gif" title="xbmc" />

... plus connu sous le nom d'[XBMC](http://xbmc.org/), est un lecteur multimédia sous licence GNU GPL.
Logiciel destiné initialement pour la console Xbox (la première), l'équipe de dév l'a porté sur les OS tels
que Mac OS X, Windows, mais aussi Ubuntu. A noter que je n'utilise pas la fonction [PVR](http://fr.wikipedia.org/wiki/Personal_Video_Recorder)
d'XBMC, puisque je n'ai pas de carte d'acquisition TV pour recevoir et enregistrer les chaines de TV directement
sur mon HTPC.

## L'installation

Il y a 3 méthodes d'installation, pour tous les goûts.

### 1. le mode fénéant

Depuis la 12.04, xmbc est dans les dépôts officiel. L'installation est donc assez simple

```shell
$ sudo apt-get install xbmc
```
*Ewh Voualaaah!* comme dirait les américains.

### 2. via [PPA](https://help.launchpad.net/Packaging/PPA)

```shell
$ sudo apt-get install python-software-properties pkg-config
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:team-xbmc/ppa
$ sudo apt-get update
$ sudo apt-get install xbmc
```

Par ailleurs, il est possible de choisir un autre répertoire, entre le ppa basic, unstable et le nightly build


### 3. la version sekshy

Une compilation des sources des plus basiques, sans trop de fioriture.

1.récupérer les sources avec git

```shell
$ git clone git://github.com/xbmc/xbmc.git xbmc
```

2.installer les dépendances

```shell
$ sudo apt-get install automake autopoint bison build-essential ccache cmake curl cvs default-jre fp-compiler
gawk gdc gettext git-core gperf libasound2-dev libass-dev libavcodec-dev libavfilter-dev libavformat-dev
libavutil-dev libbluetooth-dev libbluray-dev libbluray1 libboost-dev libboost-thread-dev libbz2-dev libcap-dev
libcdio-dev libcec-dev libcec1 libcrystalhd-dev libcrystalhd3 libcurl3 libcurl4-gnutls-dev libcwiid-dev libcwiid1
libdbus-1-dev libenca-dev libflac-dev libfontconfig-dev libfreetype6-dev libfribidi-dev libglew-dev libiso9660-dev
libjasper-dev libjpeg-dev libltdl-dev liblzo2-dev libmad0-dev libmicrohttpd-dev libmodplug-dev libmp3lame-dev
libmpeg2-4-dev libmpeg3-dev libmysqlclient-dev libnfs-dev libogg-dev libpcre3-dev libplist-dev libpng-dev
libpostproc-dev libpulse-dev libsamplerate-dev libsdl-dev libsdl-gfx1.2-dev libsdl-image1.2-dev
libsdl-mixer1.2-dev libshairport-dev libsmbclient-dev libsqlite3-dev libssh-dev libssl-dev libswscale-dev
libtiff-dev libtinyxml-dev libtool libudev-dev libusb-dev libva-dev libva-egl1 libva-tpi1 libvdpau-dev libvorbisenc2
libxml2-dev libxmu-dev libxrandr-dev libxrender-dev libxslt1-dev libxt-dev libyajl-dev mesa-utils nasm pmount
python-dev python-imaging python-sqlite swig unzip yasm zip zlib1g-dev libtag1-dev
```

3.et on compile

```shell
$ cd *répertoireXBMC_depuisGIT*/xbmc/
$ ./bootstrap
$ ./configure
$ make -j2
$ make install
```

L'option **-j2** permet de compiler les sources en utilisant la parallélisation des [jobs](http://www.blaess.fr/christophe/2012/01/14/parallelisation-de-compilations/) de votre processeur
si ce dernier supportant l'hyper-threading ou est est multi-coeurs. Dans mon cas, j'ai un dual core.

Et c'est fini pour l'install!


### Source :
+ [Installation 1](http://wiki.xbmc.org/index.php?title=Installing_XBMC_for_Linux)
+ [Installation 2](http://wiki.xbmc.org/index.php?title=HOW-TO:Compile_XBMC_for_Ubuntu)
