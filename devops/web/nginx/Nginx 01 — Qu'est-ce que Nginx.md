#devops #nginx #bases

## Qu'est-ce que Nginx ?

Nginx (prononcé *"engine x"*) est un logiciel serveur créé en 2004 par Igor Sysoev pour répondre au problème **C10K** : tenir des dizaines de milliers de connexions simultanées avec peu de ressources, là où les serveurs de l'époque (Apache en mode process/thread) s'effondraient. Il ne fait pas qu'une seule chose : c'est un logiciel multi-rôles, choisi selon la configuration qu'on lui donne.

## Les rôles que Nginx peut jouer

- **Serveur web** : sert des fichiers statiques directement depuis le disque (`root`, `index`) — HTML, CSS, JS, images.
- **Reverse proxy** : reçoit la requête du client à sa place, la transmet à un serveur backend, puis renvoie la réponse — le client ne voit jamais le backend directement.
- **Load balancer** : répartit les requêtes entre plusieurs instances backend selon un algorithme (round-robin, least_conn...).
- **Passerelle FastCGI** : transmet une requête à un processus applicatif (typiquement PHP-FPM) via le protocole **FastCGI**, pas via HTTP classique.
- **Terminaison TLS** : déchiffre le HTTPS côté serveur avant de transmettre la requête en clair au backend interne.

## Illustration — un seul fichier, plusieurs rôles combinés

```
Client (HTTPS)
    │
    ▼
┌─────────────────────────────┐
│           Nginx             │
│  • termine le TLS           │  ← rôle "terminaison TLS"
│  • sert les fichiers statiques │ ← rôle "serveur web"
│  • route les .php ──────────┼──▶ PHP-FPM (FastCGI)  ← rôle "passerelle FastCGI"
└─────────────────────────────┘
```

C'est exactement le cas de la configuration WordPress de référence de ce module : un seul bloc `server` sert de terminaison TLS, de serveur de fichiers statiques, et de passerelle vers PHP-FPM.

## Cas particuliers

> [!warning] `proxy_pass` et `fastcgi_pass` ne sont pas interchangeables
> `proxy_pass` transmet une requête HTTP complète à un serveur qui parle HTTP (une API, un autre reverse proxy...). `fastcgi_pass` utilise un protocole binaire différent, FastCGI, pour dialoguer avec un processus comme PHP-FPM. On ne peut pas faire `fastcgi_pass` vers un serveur HTTP classique, ni `proxy_pass` vers PHP-FPM.

> [!tip] Retenir Nginx par ses rôles, pas par une définition unique
> Face à une configuration inconnue, se demander d'abord *"quel(s) rôle(s) ce bloc `server` joue-t-il ?"* (web, proxy, FastCGI, load balancer) est plus utile que de chercher une définition générale de Nginx.
