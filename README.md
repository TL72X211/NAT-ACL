# NAT-ACL
mots clés :

NAT dynamique
surcharge NAT
pool adresses publiques
même adresses publiques
requêtes WEB
adressage privé

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
