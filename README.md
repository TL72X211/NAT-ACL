

# NAT-ACL
mots clés :

**NAT dynamique :** plusieurs @IP publique & privés (cf en bas)

**surcharge NAT :**  C'est un PAT

**pool adresses publiques** = piscine d'@ publiques = plusieurs @IP publiques réservés.

**même adresses publiques :** /

**requêtes WEB** : /

**adressage privé** : Qui est unique sur le réseau local.

contexte :
pourquoi ? fournir plusieurs adresses publiques / add privée -> publique

Quoi ?	configurer NAT dynamique / rediriger sur une add publique une application
comment ? en utilisant NAT

contraintes :

utiliser le pool d’adresses IP fournis
problématique :

Comment transcrire plusieurs add privées en une adresse publique en configurant un NAT dynamique ?

généralisation :

changement de référentiel
MCO

Hypothèses :

On peut rediriger les req web avec le socket
Cmd ip nat pool ip1 ip2 -> range
Cmd fonctionnent pas car par dans le mode de configuration
utiliser une wilde card netmask

plan d’action :

etude:

étudier le NAT (statique/dynamique)
étudier PAT
ACL
cmds de configs (prosit)

réalisation:

packet tracer


## 1 - NAT : Translation d'adresses - Principes de bases  

Pour parier au manque d'@IPv4 Libres, ce sont des @codés sur 4 octets, le principe est de remplacer à la volée les champs d'adresses privés dans les paquets qui sont destinés à un autre réseau.

On retrouve dans le NAT quatre types d'adresses :
- **Inside local address** : @IP assignée à un hôte à l'intérieur d'un réseau d'extrémité. Il s'agit d'une @IP privée
- **Inside global address** : La ou les @IP publiques qui représentent les @IP locales internes, dites "routables" du NAT.
- **Outside local address** : @IP d'un hôte telle qu'elle apparaît aux hôtes d'un réseau internet. Il ne s'agit pas nécessairement d'une @ légimite routable.
- **Outside global address** : @IP réeele routable d'un hôte qui se situe à l'extérieur du réseau du routeur NAT.

![](https://blog.nicolargo.com/wp-content/uploads/2011/07/NAT.png)


**NAT BASIQUE**

Attribuer de façon automatique une @IP à une autre. Il suffit de modifier le paquet, en ayant le même nombre d'IP ext que d'IP privés.1 @privée avec 1@publique.

**NAT dynamique**

Il peut y avoir plusieurs IP extérieures tout comme plusieurs IP privées. Cela entraine nécessairement un suivi de la connexion, car NAT attribue l'IP ext lors d'une requête qui provient d'un réseau privé.
A l'inverse, il doit pouvoir "discriminer" les paquets lors de  la réception pour attribuer à une @IP sur un réseau privé --> Problème : On est limités au réception sur le réseau local par le nbr d'adresses IP publique que l'on dispose pour traiter simultanément.
Comme basique, mais on pioche sur une pool.

**NAT MASQ** (Network Address and Port Translation) :

Permet de résoudre le problème du NAT dynamique, le nbr d'adresses externes est illimité. Un nouveau problème se pose alors :  "A quelle machine est destinée un paquet entrant, puisqu'ils ont la même @IP  de destination" (Gateway).

Pour se faire, NAT concerne une trace plus complète des paramètres de chaque connexion :
- @Source (La majorité du temps, les paquets viennent de l'interne vers l'externe), permettant d'identifier la machine à l'origine.
- Protocole supérieur : Regarder les protocoles de transport (couche 4), en fonction de l'utilité, NAT peut les catégoriser et les reconnaître.
- Le port : Grâce aux n° de ports, le NAT pourra faire la différence entre des paquets entrants qui représentent la même IP source, le même protocole de transport, mais un n° de port différent.

Si les 3 infos sont les même, le NAT éffectue une translation de port & d'adresse pour identifier de manière certaine : Cela consiste à modifier les paramètres de la connexion avec la machine distance de façon à utiliser le port voulu sur la passerelle ou se situe le NAT.

C'est appellé MASQ car faire cette opération est comparable à une attaque du type "man in the middle".

**NAPT** est identique à la NAT MASQ, sauf qu'il présente des services de redirection des flux entrant ou sortants. Ainsi, le Port Forwarding permet à l'extérieur d'accéder à un service (serveur WEB...) qui est une machine du réseau privé. On peut aussi rediriger de l'autre côté, via le "redirect" vers des proxies, des firewall...

**Bi-Directional NAT**

A la différence des deux précedents, il permet à des machines distantes d'accéder à des machines du réseau privé (contrairement au Port Forwarding) qui se limitte aux services. La passerelle répond par sa propre @IP tout en gardant en mémoire l'association entre l'@IP distante et @IP requise pour le service, ainsi les paquets sont transférés vers la machine sur le réseau local.
Le problème vient du service DNS qui peut-être couteux (peu sécurisé), cependant, c'est utile si c'est pour connecter plusieurs réseaux privés car les serveurs DNS sont mieux maîtrisés.

**Le Twice-NAT**

Technique de double translation d'adresses et de port, avec ceux de destination et ceux de la source qui seront modifiés. Le NAT cache les @Internes et les @externes vis à vis du réseau privé.
Utile quand plusieurs réseaux privés sont interconnectés, il permet de résoudre les problèmes de collisions en modifiant les 2 @ du paquet (quand plusieurs machines utilises la même @privé)

**NAT avec Serveur virtuel / Load Balancing**

On a une machine virtuelle qui sert de NAT virtuel, a cette machine est associée plusieurs machines réelles. 
Quand une nouvelle machine se connecte, elle passera par ce NAT virtuel, et utilisera une des @IP des machines réelle pour se connecter sur internet. 	C'est le NAT qui s'occupe de faire la translation, qui trasnmet la requête et la connexion associée, il sert de passerelle.

La sélection de l'adresse réelle peut se faire sur la base de la charge de travail de la machine correspondante, et si le NAT est surcharé ,on peut prendre un autre NAT moins surchargé (une autre machine non réelle).

NB : Il est également possible d'utiliser des routes virtuelles sur le NAT est sur le routeur, la passerelle possédant plusieurs interfaces, il peut choisir laquelle il souhaite en fonction de la charge de trafic.

## 2 - Avantages /  Inconvénients   


- Economiser les @IP
- Interconnecter plusieurs réseaux privés de façon transparente (même si conflit)
- Dans la majorité du temps, les machines externes ne verront que l'@ de passerelle.


## 3 - Sécurités & NAT :

- Casse tout le contrôle d'intégrité au niveau IP et au niveau Transport (@ dans checksum) (Il doit donc les recalculer)ii
- Protocoles IP comme IPSec incompatible avec le NAT
- Impossible de chiffrer les infos puisque NAT les utilises (couches "hautes").

- NAT peut protéger les machines du r@ privés vu qu'elles ne sont pas accéssibles de l'extérieur
- Prémunir contre un monitoring du traffic qui viserait à scruter les com' entre 2 machines


## 4 - Configurer son NAT :
https://www.ciscomadesimple.be/2013/04/06/configuration-du-nat-sur-un-routeur-cisco/

*$ip nat inside* ou ip nat outside --> à indiquer sur chaque interface indiquant si on est à l'intérieur ou à l'extérieur du réseau.


**NAT STATIQUE**

*R1(config)#ip nat inside source static 192.168.1.100 201.49.10.30*

*R1#show ip nat translations*  -> Vérifier la config

*SP#debug ip packet* -> Vérifier les paquets qui passent par le router 

**Avec une pool d'adresse**

*R1(config)#ip nat pool POOL-NAT-LAN2 201.49.10.17 201.49.10.30 netmask 255.255.255.240* -> Tranche d'adresses

--> Gestion des deny

*R1(config)#access-list 1 deny 192.168.1.100*

*R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255*

-> Configurer le NAT lui même 

*R1(config)#ip nat inside source list 1 pool POOL-NAT-LAN2* 

"On instruit donc ici le routeur de créer dynamiquement une translation pour les paquets arrivant sur une interface « inside » routés par une interface « outside » dont l’adresse IP source correspond à l’ACL 1 et de remplacer l’IP source par une de celles comprises dans le pool POOL-NAT-LAN2."

*Pour toute adresse IP privée qui n'est pas comprise dans la pool, on utilise le mot overload, ça passe en PAT, et on fait donc une surcharge c'est à dire que 2 postes partagent la même IP, sans oublier de translater les n° de port*

**Configuration du NAT dynamique avec surcharge (sans pool)**

-> Gestion des deny

*R1(config)#access-list 2 permit 192.168.0.0 0.0.0.255*

-> On fait tout passer en overload

*R1(config)#ip nat inside source list 2 interface serial 0/0 overload*

-> debug avec debug ip packets

## 5 - Qu'est ce qu'un PAT :

![](https://blog.nicolargo.com/wp-content/uploads/2011/07/PAT.png)

1 @ IP publique
5 @ Privées

Quand une des @ envoie un requête vers le port 1044, le NAT :
OK je convertit vers une @ + n° de PORT

Si deux @ demande le même port, le deuxième aura n°port +1



