#devops #nginx #architecture

## Master et workers

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}
```

Nginx démarre toujours avec un **master process** (souvent lancé en root, pour pouvoir écouter sur les ports privilégiés comme 80/443) qui ne traite aucune requête lui-même. Il lit la configuration, ouvre les sockets d'écoute, puis lance un ou plusieurs **worker processes** — ce sont eux qui traitent réellement les connexions.

## Rôle de chaque directive

| Directive | Rôle |
|-----------|------|
| `user www-data` | Les workers tournent avec les privilèges de `www-data`, pas root — le master reste root uniquement pour biner les ports privilégiés, principe du moindre privilège |
| `worker_processes auto` | Nginx détecte le nombre de cœurs CPU disponibles et lance un worker par cœur |
| `pid /run/nginx.pid` | Fichier où le PID du master est stocké, utilisé par les scripts d'administration (`nginx -s reload`, systemd...) |
| `worker_connections 1024` | Nombre maximum de connexions simultanées qu'**un seul worker** peut gérer |

## Capacité totale du serveur

```
clients_max ≈ worker_processes × worker_connections
```

> [!warning] `worker_connections` compte aussi les connexions vers les backends
> Si un worker fait office de reverse proxy, chaque requête client ouvre potentiellement une deuxième connexion (vers le backend). Dans ce cas, la capacité réelle en clients simultanés peut descendre à environ `worker_connections / 2`.

> [!tip] Reload sans coupure
> Un `nginx -s reload` (ou `SIGHUP` envoyé au master) recharge la configuration en démarrant de nouveaux workers avec la nouvelle config, puis termine proprement les anciens une fois leurs connexions en cours terminées — aucune requête n'est coupée pendant le changement.
