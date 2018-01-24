# NAT-ACL

## Mots clés

 * NAT dynamique : Network Address Translation
 * Surcharge NAT : PAT, 
 * Pool adresses publiques : 
 * Même adresses publiques : 
 * Requêtes WEB : 
 * Adressage privé : 

## Contexte
### Pourquoi ?
Fournir plusieurs adresses publiques / add privée -> publique

### Quoi ?
Configurer NAT dynamique / rediriger sur une add publique une application

### Comment ?
En utilisant NAT

## Contraintes
 * Utiliser le pool d’adresses IP fournis

## Problématique
Comment transcrire plusieurs add privées en une adresse publique en configurant un NAT dynamique ?

## Généralisation
 * Changement de référentiel
 * MCO

## Hypothèses
 * On peut rediriger les req web avec le socket
 * Cmd ip nat pool ip1 ip2 -> range
 * Cmd fonctionnent pas car pas dans le bon mode de configuration
 * Utiliser une wilde card netmask

## Plan d’action
### Etude

 * Etudier le NAT (statique/dynamique)
 	 * NAT Statique : NAT basique, attribue de façon automatique une adresse IP à une autre. Il suffit de modifier l'adresse du paquet suivant la règle définie. dans ce cas là l'idéal est d'avoir autant d'adresse privées que publiques.

 	 * NAT Dynamique : N'a aucune association préféfinie entre une IP publique et privée (pool disponible), qu'il attribue lors de la connexion pour ensuite discriminer les paquets en fonction de l'IP pour rester transparent vis à vis de l'hôte distant et local. Un problème survient si l'on a pas assez de clé publique car plus aucun équipement supplémentaire ne pourra communiquer avec le réseau extérieur.

 	 * NAPT Masq (Network Address and Port Translation) : Permet de résoudre les problèmes de deux NAT précédents même lorsque le nombre d'adresses externe est limité (fonctionne pour plusieurs équipements partageant la même IP publique). C'est possible grâce au 'contexte' de chaque connexion, le NAPT tient compte :
 	 	 * De l'adresse source : chaque équipement du réseau aura tendance à communiquer avec une machine différentes des autres ce qui permet de les différencier
 	 	 * Du protocole supérieur employé : TCP ou UDP par exemple
 	 	 * Du port : différencie deux paquets ayant la même IP source mais deux ports de destination différents
 	 Lorsque toutes ces informations ne suffisent pas à différencier les communications le NAT peut effectuer une translation de port pour identifier les deux différents destinataires, ce qui reste invisible aux yeux des utilisateurs.

 	 * NAPT Redirect / Port Forwarding : Identique au NAPT Masq et permet également de faire croire à la machine distante qu'elle communique directement avec un serveur qui est en fait en privé et elle passe donc d'abord par le NAT. Le redirect permet de rediriger un flux sortant vers des services particuliers (proxy, firewall, ..)
 	 	* Inside local
 	 	* Inside global
 	 	* Outside global
 	 	* Outside local
 	 * Bi-directional NAT : Diffère des précédents, il permet à des machines distantes d'accéder à des machines du réseau privé directement. Le principe fait appel au service DNS pour interpréter les requêtes, elles sont initiées par la machine distante et reçues par le NAT. La passerelle répond par sa propre adresse IP tout en gardant en mémoire l'association entre l'IP distante et l'IP requise pour le service. Ainsi, les paquets provenant de la machine distante seront transférés vers la machine correspondante directement. Le problème de cette technique est l'utilisation du service DNS qui peut être couteux dans le cas d'un utilisateur de base accédant au réseau public Internet mais qui par contre pourra se révéler utile dans le cas d'une entreprise interconnectant plusieurs réseaux privés car les serveurs DNS sont alors mieux maitrisés. 

 	 * Twice-NAT : Technique de double translation d'adresses et de ports. A la fois les paramètres de destination et ceux de la source seront modifiés. Le NAT cache les adresses internes vis-à-vis de l'extérieur ainsi que les adresses externes vis-à-vis du réseau privé. L'utilité de cette technique apparait quand plusieurs réseaux privés sont interconnectés : Le Twice-NAT permet de résoudre les problèmes de machines utilisant des plages d'adressage bien précises qui peuvent créer des conflits et des collisions entre plusieurs réseaux privés (c'est-à-dire plusieurs machines utilisant la même adresse IP privée) en modifiant les 2 adresses du paquet. 

 	 * NAT Serveurs Virtuels - load Balancing : Evolution des techniques de NAT qui permet d'optimiser leurs implémentations. L'utilisation de serveurs virtuels est actuellement très répandue; cela correspond à une machine inexistante représentée uniquement par son adresse IP et prise en charge par une ou plusieurs machines réelles qui ont également leurs propres adresses (différentes). Ainsi, les requêtes des machines distantes sont adressées à une ou plusieurs adresses virtuelles correspondant à la passerelle où est implémenté le deamon effectuant le NAT. Celui-ci remplace alors l'adresse virtuelle par une des adresses réelles appartenant aux machines implémentant le service NAT, puis leur transmet la requete et la connexion associée.
 	 La sélection de l'adresse réelle peut se faire sur la base de la charge de travail de la machine correspondante: si le serveur NAT est surchargé, on choisira un autre serveur NAT moins chargé. Cette technique est à la base du load balancing et il existe de nombreux algorithmes de sélection et de répartition de la charge.
     Enfin, comme le service NAT est habituellement placé sur la machine chargée du routage, une évolution possible est l'utilisation de routes virtuelles tout comme nous avons vu les adresses virtuelles. Dans ce dernier cas, la passerelle possède plusieurs interfaces vers le réseau externe et peut choisir laquelle utiliser en fonction de la charge de traffic sur chaque brin.
cmd nat statique : en config "ip nat inside static *adresse.interne adresse.externe*"
cmd nat dynamique : en config "access-list *int liste d'accès* permit *source* *masque générique*
								ip nat pool *int plage* *adresse de départ plage* *adresse fin plage* netmask *masque de sous réseau*
								ip nat inside source list *int liste d'accès* pool *int plage ip*"

cmd pat : en config "acces-list *numero de la liste* permit *ip source* *masque générique ex:0.0.0.255*
								ip nat inside source list *numéro de la liste* interface *type et numéro d'interface ex:f0/1* overload
								**définir les interfaces inside et outside**"

vérification : "show ip nat translation"


 * ACL (Access Control List)
 	 Liste permettant de filtrer les paquets IP, elles sont lues de manière séquentielle par les routeurs, la première instruction rencontrée pour le paquet en question qui fait foi, et les instructions suivantes ne sont pas utilisées.  
 	 Correspondance des codes :
 	 Codes ACL : 
	 RouterAidoweb(config)# access-list ?  
	 <1-99> IP standard access list  
	 <100-199> IP extended access list  
	 <200-299> Protocol type-code access list  
	 <300-399> DECnet access list  
	 <600-699> Appletalk access list  
	 <700-799> 48-bit MAC address access list  
	 <800-899> IPX standard access list  
	 <900-999> IPX extended access list  
	 <1000-1099> IPX SAP access list  
	 <1100-1199> Extended 48-bit MAC address access list  
	 <1200-1299> IPX summary address access list  
	 <1300-1999> IP standard access list (expanded range)  
	 <2000-2699> IP extended access list (expanded range)  

	 Commandes ACL :  
	 	 * Définition d'une ACL Standard : **RouterAidoweb(config)# access-list 20 permit 10.2.128.0 0.0.15.255**  
	 	 On va accepter les paquets venant de la plage 10.2.128.0 / 10.2.143.255. Et le numéro 20 n'est que le numéro que l'on a donné à l'ACL, compris entre 1 et 99 (cf codes ACL), ou entre 1300 et 1999 (car c'est une ACL standard, sur le protocole IP)

	 	 * Définition d'une ACL Etendue : **RouterAidoweb(config)# access-list 121 deny tcp host 10.2.16.1 10.2.128.0 0.0.15.255 eq 23**  
	 	 Ici on interdit à l'hôte 10.2.16.1 l'accès telnet (car port TCP 23) au réseau 10.2.128.0/20. La première IP concerne l'hôte (défini par host), la seconde le réseau destination, et la troisième est le masque générique

	 	 * Application de l'ACL à une interface : 

 * Cmds de configs (prosit)
 	 * ip nat pool 205.0.0.1 205.0.0.6
 	 * ip nat inside source list 1 pool
 	 * access-list 1 permit 192.168.0.0 0.0.255.255

### Réalisation
 * Packet Tracer