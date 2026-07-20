#devops #caddy #reverse-proxy #intermédiaire

## Configuration minimale

```caddyfile
api.example.com {
    reverse_proxy localhost:3000
}
```

En une ligne, Caddy écoute sur `api.example.com` (80 et 443), obtient un certificat HTTPS, transmet les requêtes à `localhost:3000`, et ajoute automatiquement les en-têtes `Host`, `X-Forwarded-For`, `X-Forwarded-Proto` — l'équivalent des `proxy_set_header` qu'il faut déclarer explicitement chez Nginx (voir [[Nginx 10 — Reverse proxy HTTP & WebSockets]]) ou des `RequestHeader set` chez Apache (voir [[Apache 07 — Reverse proxy]]).

## Reverse proxy sur un chemin spécifique

```caddyfile
example.com {
    # L'API sur /api/*
    reverse_proxy /api/* localhost:3000
    # Le reste en statique
    root * /var/www/frontend
    file_server
}
```

## Le piège du préfixe conservé ou retiré

```caddyfile
# Sans réécriture : /api/users → backend reçoit /api/users
reverse_proxy /api/* localhost:3000

# Avec réécriture : /api/users → backend reçoit /users
handle_path /api/* {
    reverse_proxy localhost:3000
}
```

> [!warning] `handle_path` retire le préfixe, `reverse_proxy` seul le conserve
> Comportement analogue au piège du slash final dans `proxy_pass` chez Nginx (voir [[Nginx — Pièges classiques]]) — vérifier ce que le backend attend réellement avant de choisir l'une ou l'autre forme.

## WebSocket : rien à configurer

```caddyfile
example.com {
    reverse_proxy /ws/* localhost:3000
}
```

Caddy gère automatiquement l'upgrade WebSocket, sans directive dédiée — contrairement à Nginx, qui exige `proxy_http_version 1.1` et deux `proxy_set_header` explicites (voir [[Nginx 10 — Reverse proxy HTTP & WebSockets]]).

## Load balancing

```caddyfile
example.com {
    reverse_proxy {
        to backend1:3000
        to backend2:3000
        to backend3:3000
        lb_policy round_robin
        health_uri /health
        health_interval 10s
    }
}
```

| Élément | Rôle |
|---------|------|
| `to` | Ajoute un backend au pool — équivalent d'un `server` dans un bloc `upstream` Nginx (voir [[Nginx 13 — Cache, compression & load balancing]]) |
| `lb_policy` | Algorithme de répartition (`round_robin` par défaut, `least_conn`, `ip_hash`...) |
| `health_uri` / `health_interval` | Vérification active de la santé de chaque backend, retire automatiquement un backend qui ne répond plus |

> [!info] Health checks actifs par défaut avec health_uri
> Contrairement à Nginx où un `server ... backup;` reste statique jusqu'à une erreur constatée, `health_uri`/`health_interval` fait sonder Caddy périodiquement chaque backend — un membre qui répond de nouveau réintègre automatiquement le pool.

## Cas particuliers

> [!warning] Ne pas préfixer par `https://` dans reverse_proxy sauf nécessité
> `reverse_proxy https://localhost:3000` échoue souvent si le backend ne parle qu'en HTTP local — n'ajouter `https://` que si le backend exige réellement TLS en interne.

> [!tip] Ordre des handlers : du plus spécifique au plus générique
> Un handler large (`file_server` catch-all) placé avant un matcher précis (`/api/*`) capture tout avant que la règle spécifique ne soit évaluée — toujours placer les chemins précis en premier, comme pour l'ordre des `location` chez Nginx (voir [[Nginx 06 — Routing avec location & try_files]]).
