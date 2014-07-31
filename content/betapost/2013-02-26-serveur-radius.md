---
title: "Serveur Free RADIUS sur une Debian Wheezy, pour router/switch Cisco"
description: "radius, cubitus, humerus..."
date: "2013-02-26"
tags: ["linux","squeeze","cisco","radius","authentification","AAA"]
---


[RADIUS](http://fr.wikipedia.org/wiki/Remote_Authentication_Dial-In_User_Service) <del>l'os de l'avant-bras</del> (Remote Authentication Dial-In User Service) est un protocole de type C/S permettant de fournir un service centralisé d'authentification, d'autorisation et de gestion des comptes utilisateurs. 

Le but de ce billet est de pouvoir implémenter un serveur RADIUS libre, notamment sous Debian, en utilisant via le package FreeRADIUS. Le modèle de sécurité AAA (du moins pour la partie authentification et autorisation) pourra être mis en oeuvre sur les routeurs/switchs Cisco qui se synchroniseront avec le serveur RADIUS. 


## FreeRADIUS sur Debian Wheezy 7.0
-----------------------------------

### Installation des paquets:

```
# aptitude install mysql-client mysql-server freeradius freeradius-utils freeradius-mysql php5 php-pear php5-gd php-DB
```

L'installation de mysql et de php permet de gérer une base de données d'utilisateurs via une interface web, non montré dans ce billet.
Dans ce qui suit, notre base de données d'utilisateurs sera un simple fichier (ce n'est pas une _best practice_ en soi, mais je le décris ici pour des raisons de simplicités)

### Définition d'un utilisateur 
Par défaut, la configuration des utilisateurs et mots de passe associés se fait dans le fichier `/etc/freeradius/users`
Il existe plusieurs gabarits permettant de définir un utilisateur et son mot de passe associé, ainsi que les niveaux de droit qui lui sont accordés (par exemple pour administrer du matériel Cisco). L'exemple ci-dessous permet à l'utilisateur `toto` une fois authentifié d'administrer un routeur/switch Cisco avec le niveau de privilège 15:

```
toto 	Cleartext-password:="Il0v3Y0u"
	Service-Type = NAS-Prompt-User,
	cisco-avpair = "shell:priv-lvl=15"
```
__/!\ ATTENTION /!\\__ le fichier de configuration est sensible à l'__indentation__


### Test du serveur RADIUS en local
La commande type à entrer dans le shell pour tester le serveur RADIUS est la suivante : `radtest <user> <user_password> {@IP|hostname radius} <port> <radius_passwd>`:
```
# radtest toto supersecret localhost 1812 testing123
```
Et sur la sortie standard, nous avons :
Last login: Sun Jun 16 15:45:07 2013 root@localhost:~# radtest user1 supersecret localhost 1812
testing123 Sending Access-Request of id 100 to 127.0.0.1 port 1812 User-Name = "user1" UserPassword = "supersecret" NAS-IP-Address = 127.0.0.1 NAS-Port = 1812 Message-Authenticator
= 0x00000000000000000000000000000000 rad_recv: Access-Reject packet from host 127.0.0.1
port 1812, id=100, length=20
Ajout d'un utilisateur
# vim /etc/freeradius/users
toto
Cleartext-password
:=
Service-Type = NAS-Prompt-User, cisco-avpair = "shell:priv-lvl=15"

"Il0v3Y0u"

ATTENTION: la syntaxe du fichier est indent-sensitive. On ne le voit peut-être pas ci-dessus,
mais mettez une tabulation après l'username l'acronyme NAS se réfère au terme Network Access
Server, et non autre chose ;) NB: Comme vous avez pu le constater, on peut définir le niveau de privilège qu'aura
l'utilisateur ajouté par vos soins. Pareil si vous souhaitez également spécifier le type de commande autorisé pour vos
utilisateurs, avec par exemple :

cisco-avpair = "shell:cmd=show"

Il existe des manières plus granulaires de configurer l'authentification ainsi que l'autorisation d'un
utilisateur. Le fichier /etc/freeradius/users en donne des exemples. Un petit restart du serveur radius
pour vérifier que tout va bien :
# service freeradius restart

Et au pire, si ça ne va pas...
# less /var/log/freeradius/radius.log

Ajout d'un client au serveur RADIUS
Cela se passe là, on ajoute un routeur cisco, joignable par l'adresse 172.25.0.254/24 (pareil, intentsensitive syntax):
#
client
k3yStR0k3
shortname=R1
nastype=cisco}

vim
172.25.0.254

/etc/freeradius/clients.conf
{

où secret=<PSK>, shortname=<un_nom_comme_un_nom>, nastype=<NAS-specific> Pour <NASspecific>, voir le contenu du fichier clients.conf et enfin rebelotte :
# service freeradius restart

Vérification du port utilisé par RADIUS
D'après la RFC 2865, le port utilisé par RADIUS pour l'identification est le 1812 (UDP), ou encore
anciennement le port 1645. Après activation du daemon freeradius, un petit netstat pour vérifier tout
cela:
root@localhost:/var/log# netstat -patune | grep radius udp 0 0 0.0.0.0:40980 0.0.0.0:* 112 13910
9714/freeradius udp 0 0 0.0.0.0:1812 0.0.0.0:* 112 13905 9714/freeradius udp 0 0 0.0.0.0:1813
0.0.0.0:* 112 13906 9714/freeradius udp 0 0 0.0.0.0:1814 0.0.0.0:* 112 13909 9714/freeradius
udp 0 0 127.0.0.1:18120 0.0.0.0:* 112 13908 9714/freeradius
Ça m'a l'air pas trop mal!
Cisco AAA
Je ne sais pas si ce que je vais faire en dessous fait partie des best practices du constructeur. Si ce n'est pas le cas, mes excuses Cisco-

Nous utiliserons une method-list nommé AUTH-TPHO, ainsi que la liste par défaut Pour la partie
authentication de la méthode AUTH-TPHO, le routeur interrogera dans un premier temps le serveur
RADIUS. S'il n'y a aucune entrée correspondant à la paire login/mdp tapée par l'utilisateur, le routeur ira
toper sa propre base (la base locale). Enfin si l'interrogation à cette dernière base n'est pas concluante,
l'accès à ce routeur pourra se faire via le mot de passe défini par la commande enable. La méthode par
défaut interrogera dans l'ordre le serveur radius et/ou la base locale Concernant la partie authorization,
elle concernera l'exec mode, se basera sur la method-list par défaut, topera dans un premier temps le
serveur RADIUS, sinon en second la base locale, sinon en dernier enable. Enfin, on appliquera la liste
philes!

AUTH-TPHO au vty
R1#configure
Enter
configuration
commands,
one
per
R1(config)#username
adminTPHO
R1(config)#enable
secret
R1(config)#aaa
R1(config)#aaa
authentication
login
AUTH-TPHO
R1(config)#aaa
R1(config)#aaa

authentication
authorization

R1(config)#radius-server
k3yStR0k3

host

login
exec
172.25.0.1

line.
End
secret

group

radius

default
default
auth-port

terminal
CNTL/Z.
ciscoTPHO
cisc0TPHo
new-model
local
enable

with

radius
radius

local
local

1812

key

R1(config)#line vty 0 4
R1(config-line)#login authentication AUTH-TPHOR1(config-line)#transport input ssh
R1(config-line)#end
R1#copy running-config startup-config

Test
On suppose avoir activer au préalable debug aaa authentication et debug aaa authorization, pour voir si
le routeur tape bien sur le serveur Free RADIUS. On se connecte via SSH :
tpho@pwet:~$ ssh toto@172.25.0.254 Password: Il0v3Y0u R1#
Sur le routeur, on peut observer :
*Jun
16
21:04:09.410:
AAA/BIND(00000007):
Bind
i/f
*Jun 16 21:04:09.414: AAA/AUTHEN/LOGIN (00000007): Pick method list 'AUTH-TPHO'
*Jun
16
21:04:11.782:
AAA/AUTHOR/EXEC(00000007):
processing
AV
priv-lvl=15
*Jun
16
21:04:11.786:
AAA/AUTHOR/EXEC(00000007):
processing
AV
service-type=7
*Jun 16 21:04:11.786: AAA/AUTHOR/EXEC(00000007): Authorization successful

Et voilà 
### Sources:
*[1](http://wiki.freeradius.org/vendor/Cisco) 
*[2](http://doc.ubuntu-fr.org/coovachilli#installation_et_configuration_du_serveur_radius)
*Manipulations perso
Posted in:Cisco,Linux,Réseaux | Tagged:Cisco,Debian,Radius,Routeur,Wheezy | With 0 comments