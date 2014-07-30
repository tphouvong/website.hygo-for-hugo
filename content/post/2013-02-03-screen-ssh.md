---
title: "Screen - ou comment conserver les processus sans les killer en quittant une session ssh"
description: "sssshhhhhh!"
date: "2013-02-03"
tags: ["linux","screen","ssh","ps"]
---

<img class="img-center center-block" src="/img/blog/ssh.jpg" title="ssh"/>

Screen, un gestionnaire de fenêtres en mode texte (console shell), vous permet de garder les travaux/tâches en cours que vous avez déclenché sur une machine distante via ssh. Ainsi, plus besoin de garder une session ssh ouverte pour qu'un processus se termine. Parfois, il vous arrive même qu'une session ssh tombe pour x raisons (déco, etc.). Screen est LA solution à ces problèmes!

Installation sous Debian/Wheezy

```shell
$ sudo apt-get install screen
```

Utilisation : lancement de screen

```shell
$ screen
```

A partir de ce moment, on peut lancer n'importe quel processus, notamment les plus courants : wget, ftp, apt-get truc mush, etc. Si l'on veut se déconnecter, il faut se "détacher" de cette session, sans la fermer pour autant. 


Un simple __CTRL+A puis d__ pour ce faire (attention je crois que la casse a son importance ici, lire le man!). 
Vous pouvez ensuite quitter le shell.


Enfin, si vous souhaitez retourner voir les processus en cours, reconnectez-vous via ssh, et taper la
commande suivante:

```shell
$ screen -r -d
```

Les options -r et -d signifient que l'on souhaite se __ré-attacher__ à la dernier session __detached__. 
Je vous invite d'ailleurs à lire le man de Screen si vous souhaitez bidouillez plus en profondeur. RTFM comme l'on dit si bien ...

Merci à [Julien Manteau](http://www.jmanteau.fr) pour la découverte!
