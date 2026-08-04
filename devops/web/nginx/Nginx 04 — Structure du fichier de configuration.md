#devops #nginx #configuration

## Syntaxe de base

Chaque directive se termine par un point-virgule, les blocs s'ouvrent et se ferment avec des accolades, et `#` introduit un commentaire :

```nginx
# Ceci est un commentaire
worker_processes auto;

http {
    server_tokens off;          # directive simple, une valeur
    include mime.types;         # directive avec un chemin de fichier
    log_format main '$remote_addr - $status "$request"';  # guillemets si la valeur contient des espaces
}
```

> [!warning] Un point-virgule oublié fait échouer tout le fichier
> Contrairement à des langages tolérants aux erreurs de syntaxe mineures, Nginx refuse de démarrer ou de recharger sur la moindre virgule ou accolade manquante — `nginx -t` (voir plus bas) est la façon de détecter ça avant que ça ne casse un `reload` en production.

## Les contextes s'emboîtent

La configuration Nginx est organisée en **contextes** hiérarchiques : chaque bloc `{ }` en ouvre un nouveau, et une directive placée dans un contexte parent est héritée par ses enfants, sauf si elle y est redéfinie.

## Illustration

```
main (niveau fichier, hors accolades)
├── events { ... }        → tuning des connexions (worker_connections)
└── http { ... }          → tout ce qui concerne HTTP
    ├── server { ... }    → un site / une adresse virtuelle
    │   └── location { ... }  → une route à l'intérieur du site
    └── server { ... }    → un autre site (autre server_name ou autre port)
        └── location { ... }
```

```nginx
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 443 ssl;
        # ce server hérite de include/default_type définis dans http
    }
}
```

## Règles d'héritage et de portée

- Une directive définie dans `http` s'applique à tous les `server` qu'il contient, sauf si un `server` (ou une `location`) la redéfinit localement.
- Certaines directives sont valides uniquement dans un contexte précis : `worker_connections` n'existe que dans `events`, `server_name` n'existe que dans `server`.
- `include` permet de fragmenter la configuration (`mime.types`, `fastcgi_params`, `conf.d/*.conf`) plutôt que d'avoir un unique fichier monolithique — voir la convention `sites-available`/`sites-enabled` détaillée en [[Nginx 00 — Installation]].

## Le piège des directives qui ne fusionnent pas : add_header

L'héritage décrit plus haut suggère qu'une directive redéfinie dans un contexte enfant s'ajoute à celles du parent. C'est faux pour certaines directives dites *array-like* (`add_header`, `error_page`...) : **si le contexte enfant redéfinit ne serait-ce qu'une seule occurrence, toutes les occurrences héritées du parent sont silencieusement ignorées**, pas complétées.

```nginx
http {
    add_header X-Frame-Options "DENY";
    add_header X-Content-Type-Options "nosniff";

    location /api/ {
        add_header Cache-Control "no-store";
        # ❌ X-Frame-Options et X-Content-Type-Options disparaissent ici :
        # dès qu'un add_header apparaît dans /api/, ceux de http{} ne s'appliquent plus à /api/
    }
}
```

> [!warning] La documentation officielle le dit explicitement
> Le module `ngx_http_headers_module` précise que ces directives « sont héritées du niveau précédent seulement si aucun `add_header` n'est défini au niveau courant ». La solution historique est de répéter systématiquement tous les en-têtes voulus dans chaque `location` qui en redéfinit un seul.

> [!tip] add_header_inherit : la correction native depuis Nginx 1.29.3
> La directive `add_header_inherit merge;` restaure un comportement additif : les en-têtes du parent sont conservés même si l'enfant en ajoute de nouveaux. À privilégier sur les versions récentes plutôt que de dupliquer manuellement chaque en-tête dans chaque `location`.

## Cas particuliers

> [!warning] Une directive au mauvais niveau est ignorée ou provoque une erreur
> Placer `worker_connections` dans `http` au lieu de `events`, par exemple, fait échouer le test de configuration (`nginx -t`) — chaque directive appartient à un contexte précis, pas "n'importe où".

> [!tip] `nginx -t` avant tout reload
> Tester la configuration avec `nginx -t` avant `nginx -s reload` permet de détecter une erreur de syntaxe ou de contexte sans jamais interrompre le service en production.

## Toute la chaîne d'inclusion, en un seul exemple

```nginx
# /etc/nginx/nginx.conf — le point d'entrée, chargé au démarrage
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    include /etc/nginx/conf.d/*.conf;         # réglages HTTP globaux (gzip, logs...)
    include /etc/nginx/sites-enabled/*;        # un fichier par site actif (Debian/Ubuntu)
}
```

```nginx
# /etc/nginx/sites-available/monsite.conf, activé via un lien symbolique dans sites-enabled/
server {
    listen 443 ssl;
    server_name exemple.com;

    location /api/ {
        proxy_pass http://backend:3000;
    }
}
```

> [!info] Un seul fichier chargé, tout le reste inclus
> Seul `nginx.conf` est lu directement au démarrage — tout le reste (`conf.d/`, `sites-enabled/`) n'existe que parce que `nginx.conf` l'inclut explicitement. Supprimer un fichier de `sites-enabled/` sans le retirer de `nginx.conf` (ou l'inverse) n'a aucun effet tant que la ligne `include` correspondante n'a pas changé.

## Vérifier ce qui est réellement chargé

```bash
nginx -t          # Valide la syntaxe et les contextes, sans afficher le contenu
nginx -T | less   # Affiche la configuration complète, tous les include résolus
```

`nginx -T` est l'outil de référence pour diagnostiquer une directive qui semble ignorée : il montre exactement ce que Nginx a compris après fusion de tous les fichiers inclus — voir [[Nginx 16 — Dépannage]] pour l'usage complet en contexte de panne.

## Pour aller plus loin

La convention `sites-available`/`sites-enabled` vs `conf.d` selon la distribution est détaillée en [[Nginx 00 — Installation]]. Le mécanisme de `reload` sans coupure (nouveaux workers, anciens qui terminent leurs connexions) est couvert en [[Nginx 03 — Architecture & modèle de processus]].
