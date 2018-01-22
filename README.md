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
 * Cmd fonctionnent pas car par dans le mode de configuration
 * Utiliser une wilde card netmask

## Plan d’action
### Etude

 * Etudier le NAT (statique/dynamique)

 * Etudier PAT
 
 * ACL
 
 * Cmds de configs (prosit)

### Réalisation
 * Packet Tracer