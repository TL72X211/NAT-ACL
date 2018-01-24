# NAT-ACL

## Mots clés :

* NAT dynamique: association many to many d'iP publique et privé
* surcharge NAT: PAT faire passer plusieurs IP privés sur une IP publique en les différenciant par les n° de ports
* pool adresses publiques: -
* même adresses publiques: -
* requêtes WEB: -
* adressage privé: -

## Contexte :

**Pourquoi ?**
Fournir plusieurs adresses publiques / add privée -> publique

**Quoi ?**
Configurer NAT dynamique / rediriger sur une add publique une application

**Comment ?**
En utilisant NAT

## Contraintes :
Utiliser le pool d’adresses IP fournis

## Problématique :
**Comment transcrire plusieurs adresse privées en une adresse publique en configurant un NAT dynamique ?**

## Généralisation :
* Changement de référentiel
* MCO

## Hypothèses :

* On peut rediriger les requêtes web avec le socket
* Cmd ip nat pool ip1 ip2 -> range
* Cmd fonctionnent pas car par dans le mode de configuration
* Utiliser une wilde card netmask

## Plan d’action :

### Études

#### **Étudier le NAT (statique/dynamique)**

Network Address translaton, RFC 3022, pratique extrêmement courante créée pour palier au manque d'IPv4 libres. Des plages IP furent réservées comme IP privée non routables sur internet (RFC 1918)et puisqu'elles ne sont pas routables elles doivent être traduites en IP publique. Cette traduction consiste à remplacer les champs d'adresses dans les paquets destinés à un autre réseau, le NAT doit donc être fait entre les 2 interfaces réseau.

![](picsNico/nat.jpg)

Il existe 4 type d'adresses NAT :

* inside address : cela du poste qui est traduite
* outside address: cela de la destination

Ces deux sont eux même séparés en deux:

* local address : apparaît dans le réseau privée (inside network)
* global address : apparaît dans le réseau publique (outside network)

on a donc :

* inside local address: adresse de la source vue depuis le réseau local
* inside global address: adresse de la source vue sur le réseau publique
* outside global address: adresse de la destination vue sur le réseau publique
* outside local address : adresse de la déstination vue depuis le réseau local 

techniques de traduction:

déterminé par la topologie du réseau privé, le nombre de machines, le nombre d'@ IP publiques et les besoins en termes de services, d'accessibilité et de visibilité de l'extérieur.

**NAT de base(statique)**,one to one, statique, attribue de façon automatique une IP à une autre. nb d'IP publique = nb d'IP privé (pas très utile pour le problème d'IP public donc)
**NAT dynamique**,many to many, aucune association prédéfinie entre IP publique et privée. Le NAT attribue l'IP extérieure lors de la requête initiale qui provient du réseau privé,  il doit ensuite pouvoir discriminer les paquets entrants de façon à pouvoir leur attribuer à chacun l'IP correspondante sur le réseau privé, il doit donc suivre à qui il attribue quelle IP. On rencontre un problème si on n'a pas autant d'IP publiques que privés, si toutes les adresses publiques sont utilisées aucun postes supplémentaire du réseau privé ne peut accéder à internet
**PAT (Port Address Translation)**,many to one, aka overloading Utilise les ports pour différencier les connexions (avec 65536 port il y a de la marge) ce qui lui permet de traduire plusieurs ip privée en une même adresse publique. Il existe différents type de PAT. PAT traite différemment les paquets ne possédant pas de numéro de port, ex ICMPv4. Il traite chaque protocole différemment, par exemple ICMPv4 utilise un ID de requête qui s'incrément e à chaque requête t PAT utilise cet ID
 
**NAPT MASQ**, Network address ans Port Translation permet de résoudre le problème de bijection IP privée/publique, il permet à plusieurs machines de partager une IP publique.
Pour savoir à quelle machine privée les paquets reçus sont destinés NAT conserve une trace plus complète des paramètres de chaque connexion :

* adresse source, chaque machine de réseau privée communique avec une machine extérieure différente, les paquets entrant permettent donc au NAT d'identifier la machine d'origine, l'utilisation de ce critère seul ne fonctionne pas si plus d'une machine sur le réseau privé communique avec la même machine
* le protocole supérieur
* le port 

il est cependant possible que l'IP source, le protocole de transport et le port soient les mêmes, dans ce cas le NAT effectue une translation de port en même temps que d'dresse pour être certain d'identifier les flux, cela consiste à modifier les paramètres de connexion avec la machine distante de façon à utiliser le port voulu sur la passerelle où se situe le NAT.  il existe 65535 ports disponibles (moins les 1024 réservés) ce qui est amplement suffisant.

MASQ provient du fait que cette opération est comparable à une attaque du type man-in-the-middle sauf qu'elle ne vise pas à obtenir quelqu'information que ce soit

**NAPT Redirect/Port Forwarding** comme le NAPT MASQ mais avec des services de redirection des flux entrants ou sortants. le Port Forwarding permet à l'extérieur d'accéder à un service (serveur WEB ou autre) qui est en fait basé sur une machine de réseau privé : la machine distante pense communiquer avec la machine hébergeant le NAT alors qu'en fait celui-ci redirige le flux vers la machine correspondant réellement à ce service.

**Bi-directional NAT** permet à des machines distantes d'accéder à des machines du réseau privé, et ce directement contrairement au Port forwarding.
Le principe fait appel au service DNS pour interpréter les requêtes; celles-ci sont initiées par la machine distante et reçues par le NAT. La passerelle répond par sa propre IP tout en gardant en mémoire l'association entre l'IP distante et l'IP requise pour le service, les paquets provenant de la machine distante seront alors transférés vers la machine correspondante.
Pb: utilise DNS qui peut être coûteux pour un utilisateur de base mais se révèle utilise pour une companie interconnectant plusieurs réseaux privés

**Twice NAT** double traduction d'adresses et de ports, les paramètres de destination et de source sont modifiés. utile pour la connexion entre 2 réseaux privés

**NAT avec Serveurs Virtuels/ Load balancing** évolution des techniques NAT qui permet d'opti leurs implémentations. Une machine virtuelle représentée uniquement par sont IP qui est prise en charge par une ou plusieurs machines réelles ayant leurs propres adresses.  Les requêtes des machines distantes sont adressées à une ou plusieurs adresses virtuelles correspondant à la passerelle où est implémenté le deamon NAT qui va remplacer l'adresse virtuelle par une des adresses réelles des machines implémentant le service NAT
La sélection de l'adresse réelle se fait sur la base de la charge de travail de la machine correspondante, si le serv NAT est surchargé on en choisira un autre moins chargé.
Puisque le service NAT est généralement placé sur la machine chargée du routage une évolution possible est l'utilisation de routes virtuelles, dans ce cas la passerelle possède plusieurs interfaces vers le réseau externe et peut choisir laquelle utiliser selon la charge de trafic sur chaque brin

**Problèmes de sécurité liés au NAT**

Même si il est fait pour être transparent NAT modifie les paquets IP ce qui à pour conséquence de briser tout contrôle d'intégrité au niveau IP et supérieurs puisque TCP inclue les adresses dans ses checksums. Certains protocoles comme IPSec sont incompatibles avec NAT
Une faille est qu'un NAT évolué a tendance à remonter les couches pour étudier les protocoles de transport afin de rassembler assez d'informations pour chaque contexte. Tout chiffrement à ce niveau empêcherait donc le NAT de fonctionner, puisque les informations seraient alors cryptées.

Cependant NAT permet de protéger les machines du réseau privé d'attaques directes puisqu'elles ne sont en fait pas accessibles de l'extérieur. Cela permet également de se prémunir contre un monitoring du trafic qui viserait à scruter les communications entre 2 machines particulières

Avantages et inconvénients du NAT:

av:

* économiser des IP publiques
* permet d'interconnecter plusieurs réseaux privés de façon transparente même s'il existe des conflits d'adressage ente eux
* empêche le monitoring

inc:

* les machines externes ne verront que l'adresse de la passerelle et ne pourront pas se co directement aux machines locales (résolu avec les techniques plus évoluées mais elles sont + coûteuses et peu accessibles)
* pb de sécurité
* diminution des perf (ré-encapluse L3 voir L4, recalcule les checksums)


### **ACL**

 ip sources qu'on autorise à être traduit

	R1(config)#access-list 1 deny 192.168.1.100
	R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255


### **cmds de configs (prosit)**

config un NAT:

première chose à faire : indiquer au routeur l(es) interface(s) d'entrée et celle(s) de sortie du réseau privé 
	
	R1(config-if)#ip nat inside
	R1(config-if)#ip nat outside

config statique : on indique au NAT que l'adresse privée doit être traduite par la publique à la sortie et l'inverse à l'entrée 

	R1(config)#ip nat inside source static *local-ip* *global-ip*

check la table NAT (le log)

	R1#show ip nat translations

NAT avec pool d'adresses (dynamique):

	R1(config)#ip nat pool *name* *start-ip* *end-ip* netmask *mask*

on créé une plage d'adresse nommée *name* utilisant les adresses de la range

on défini ensuite les ip sources qu'on peut traduire (on créé l'ACL)

	R1(config)#access-list 1 deny 192.168.1.100
	R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255

ici on autorise le pool 192.168.1.0/24 moins 192.168.1.100

on lie ensuite l'ACL au pool

	R1(config)#ip nat inside source list *list number* pool *pool-name*
on config le NAT en indiquant d'utiliser les IP du pool *pool-name* pour traduire les adresses de l'ACL *ACL number*
/!\ si il y a plus de machines privées que d'ip publiques il faut ajouter "overload" à la in de la commande, cela permet d'activer le PAT

on identifie ensuite les interfaces:

	interface *interface* ip nat inside|outside

config NAT dynamique sans pool:

identifier les @ sources (créer l'ACL) 
	
	R1(config)#access-list 2 permit 192.168.0.0 0.0.0.255

config le NAT pour traduire les adresse de l'ACL et les remplacer par l'IP de s'interface serial 0/0

	R1(config)#ip nat inside source list 2 interface serial 0/0 overload


pour reset tout:

	R1#clear ip nat translation *

## Réalisation:

### **Packet tracer (WS)**
