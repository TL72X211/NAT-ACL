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

## Etudier le NAT (statique/dynamique)

### Statique :
Le NAT statique permet d’attribuer autant d’adresses ip publique qu’il y a d’adresses ip privée.
Il y a 3 types de NAT :

- NAT statique unidirectionnel qui transfèrent uniquement les connexions de l'extérieur vers l'intérieur (attention, les paquets de retour sont aussi transférés). Le plus souvent lorsque la machine interne initie une connexion vers l'extérieur la connexion est traduite par une autre NAT dynamique.
- NAT statique bidirectionnel qui transfèrent les connexions dans les deux sens.
- NAT statique PAT (Port Address Translation du port serveur). Conjonction d'une NAT statique uni ou bidirectionnelle et d'une traduction du port serveur. Le nom PAT vient du fait que le port serveur/destination est transféré ; à ne pas confondre avec la NAT dynamique PAT.

### Dynamique :
Le NAT dynamique utilise moins d’adresse ip publique que le NAT statique. Il possède une liste d’adresse ip publique dans laquelle il peut se servir. Lors de la création d’une connexion il attribue une adresse ip publique à la machine dans le domaine privée. Ce sont les numéros de ports qui vont permettre d'identifier la traduction en place : le numéro du port source (celui de la machine interne) va être modifié par le routeur. Il va s'en servir pour identifier la machine interne.
Il existe 4 NAT dynamiques :

- NAT dynamique PAT (Port Address Translation du port client/source) où les adresses externes sont indifférentes (le plus souvent la plage d'adresses que votre fournisseur d'accès vous a attribuée). Le nom PAT vient du fait que le port source est modifié.
- Masquerading où l'adresse IP du routeur est seule utilisée comme adresse externe. Le masquerading est donc un sous-cas de la dynamique PAT.
- NAT pool de source est la plus vieille des NAT. La première connexion venant de l'intérieur prend la première adresse externe, la suivante la seconde, jusqu'à ce qu'il n'y ait plus d'adresse externe. Dans ce cas exceptionnel le port source n'est pas modifié. Ce type de NAT n'est plus utilisé.
- NAT pool de destination permet de faire de la répartition de charge entre plusieurs serveurs. Peu d'adresses externes sont donc associées avec les adresses internes des serveurs. Le pare-feu se débrouille pour répartir les connexions entre les différents serveurs.

## Etudier PAT

PAT (surcharge) divise les ports disponibles par adresse IP globale en trois plages : 0-511, 512-1023 et 1024-65535. PAT assigne un seul port source à chaque session UDP ou TCP. Elle essaie d'assigner la valeur de port de la demande d'origine, mais si le port source d'origine est déjà utilisé, elle parcourt la plage de ports spécifique à partir de son début pour trouver pour le premier port disponible et assigne ce dernier à la conversation.

## ACL

Une ACL ou « Access Control List » est une liste de ports et d’adresses qui sont autorisées ou non par le filtrage. On trouve les ACL sur les routeurs et pare-feu. Il existe 3 grands types d’ACL :

- L’ACL Standard : elle ne contrôle que l’adresse IP source et de destination
- L’ACL étendue : elle contrôle l’adresse IP de destination, le type de protocole, le porte source, le port de destination, les flux TCP/IP, le type de service et les priorités
- L’ACL nommée-étendue est une ACL étendue à laquelle on a affecté un nom.

ACL est intéressant d’implanter lorsque le réseau n’utilise que des ports fixes comme avec les protocoles de messageries. En revanche il vaut mieux éviter de l’utiliser lorsque les protocoles utilisent des ports qui peuvent varier.
Iptables sert à créer des pare-feu et des ACL


# mds de configs (prosit)

# réalisation:

# packet tracer
