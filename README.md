# NAT-ACL

## Mots clés

 * NAT dynamique : Network Address Translation
 * Surcharge NAT : 
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

 	 * NAT Dynamique : N'a aucune association préféfinie entre une IP publique et privée, qu'il attribue lors de la connexion pour ensuite discriminer les paquets en fonction de l'IP pour rester transparent vis à vis de l'hôte distant et local. Un problème survient si l'on a pas assez de clé publique car plus aucun équipement supplémentaire ne pourra communiquer avec le réseua extérieur.

 	 * NAPT Masq (Network Address and Port Translation) : Permet de résoudre les problèmes de deux NAT précédents même lorsque le nombre d'adresses externe est limité (fonctionne pour plusieurs équipements partageant la même IP publique). C'est possible grâce au 'contexte' de chaque connexion, le NAPT tient compte :
 	 	 * De l'adresse source : chaque équipement du réseau aura tendance à communiquer avec une machine différentes des autres ce qui permet de les différencier
 	 	 * Du protocole supérieur employé : TCP ou UDP par exemple
 	 	 * Du port : différencie deux paquets ayant la même IP source mais deux ports de destination différents
 	 Lorsque toutes ces informations ne suffisent pas à différencier les communications le NAT peut effectuer une translation de port pour identifier les deux différents destinataires, ce qui reste invisible aux yeux des utilisateurs.

 	 * NAPT Redirect / Port Forwarding : Identique au NAPT Masq et permet également de faire croire à la machine distante qu'elle communique directement avec un serveur qui est en fait en privé et elle passe donc d'abord par le NAT. Le redirect permet de rediriger un flux sortant vers des services particuliers (proxy, firewall, ..)

 	 * Bi-directional NAT :

 	 * Twice-NAT :

 	 * NAT Serveurs Virtuels - load Balancing :


 * Etudier PAT
 
 * ACL
 
 * Cmds de configs (prosit)

### Réalisation
 * Packet Tracer