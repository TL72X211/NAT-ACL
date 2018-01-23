
# NAT-ACL
mots clés :

**NAT dynamique :** plusieurs @IP publique & privés (cf en bas)
**surcharge NAT :**  Il s'agit d'une entrée de table de traduction qui contient des informations d'@IP et des ports de source/ de destination, appellée PAT ou surcharge. PAT, c'est quand le FAI fournit plusieurs @publiques, on l'utilise pour configurer et utiliser un pool. On divise les ports par @IP globales en différentes plages.
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

Pour parier au manque d'@ IP Libres, ce sont des @codés sur 4 octets, le principe est de remplacer à la volée les champs d'adresses privés dans les paquets qui sont destinés à un autre réseau.

On retrouve dans le NAT quatre types d'adresses :
- **Inside local address** : @IP assignée à un hôte à l'intérieur d'un réseau d'extrémité. Il s'agit d'une @IP privée
- **Inside global address** : La ou les @IP publiques qui représentent les @IP locales internes, dites "routables" du NAT.
- **Outside local address** : @IP d'un hôte telle qu'elle apparaît aux hôtes d'un réseau internet. Il ne s'agit pas nécessairement d'une @ légimite routable.
- **Outside global address** : @IP réeele routable d'un hôte qui se situe à l'extérieur du réseau du routeur NAT.


**NAT BASIQUE**

Attribuer de façon automatique une @IP à une autre. Il suffit de modifier le paquet, en ayant le même nombre d'IP ext que d'IP privés.

**NAT dynamique**

Il peut y avoir plusieurs IP extérieures tout comme plusieurs IP privées. Cela entraine nécessairement un suivi de la connexion, car NAT attribue l'IP ext lors d'une requête qui provient d'un réseau privé.
A l'inverse, il doit pouvoir "discriminer" les paquets lors de  la réception pour attribuer à une @IP sur un réseau privé --> Problème : On est limités au réception sur le réseau local par le nbr d'adresses IP publique que l'on dispose pour traiter simultanément.

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

Évolution des techniques de NAT qui permet d'optimiser leurs implémentations. 

On fait correspondre une machine inexistante, (sur une seule IP) par plusieurs machines réelles, qui ont leurs propres adresses.
Ainsi les requêtes reçus sont adressés à un ou plusieurs @ correspondant à la passerelle. C'est donc la machine non réelle, où est située le NAT qui remplacera l'adresse virtuelle par réelle des machines qui l'implémente, qui transmet la requête et la connexion associée.

LA sélection de l'adresse réelle peut se faire sur la base de la charge de travail de la machine correspondante, et si le NAT est surcharé ,on peut prendre un autre NAT moins surchargé (une autre machine non réelle).

NB : Il est également possible d'utiliser des routes virtuelles sir le NAT est sur le routeur, la pasaserelle possédant plusieurs interfaces, il peut choisir laquelle il souhaite en fonction de la charge de trafic.

## 2 - Avantages /  Inconvénients   


- Economiser les @IP
- Interconnecter plusieurs réseaux privés de façon transparente (même si conflit)
- Dans la majorité du temps, les machines externes ne verront que l'@ de passerelle.


## 3 - Sécurités & NAT :

- Casse tout le contrôle d'intégrité au niveau IP et au niveau Transport (@ dans checksum)
- Protocoles IP comme IPSec incompatible avec le NAT
- Impossible de chiffrer les infos puisque NAT les utilises (couches "hautes").

- NAT peut protéger les machines du r@ privés vu qu'elles ne sont pas accéssibles de l'extérieur
- Prémunir contre un monitoring du traffic qui viserait à scruter les com' entre 2 machines


## 4 - Configurer son NAT :
https://www.ciscomadesimple.be/2013/04/06/configuration-du-nat-sur-un-routeur-cisco/

*$ip nat inside*

*$ip nat pool*

{...}