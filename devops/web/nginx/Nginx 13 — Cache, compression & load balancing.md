#devops #nginx #performance #avancé

## Compression gzip

Réduit la taille des réponses transmises sur le réseau, typiquement de 60 à 80% pour du contenu textuel.

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/javascript application/json
               application/javascript application/xml image/svg+xml;
}
```

| Directive | Rôle |
|-----------|------|
| `gzip_vary` | Ajoute l'en-tête `Vary: Accept-Encoding`, nécessaire pour un cache correct en amont (CDN, navigateur) |
| `gzip_min_length` | N'active la compression qu'au-delà de cette taille — inutile (voire contre-productif) sur de très petits fichiers |
| `gzip_types` | Types MIME à compresser — les formats déjà compressés (images JPEG, vidéos) n'y figurent jamais |

## Cache pour reverse proxy

Nginx peut stocker les réponses d'un backend pour réduire sa charge, sans que chaque requête n'atteigne l'application.

```nginx
# Dans http {} : définir la zone de cache
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m
                  max_size=1g inactive=60m use_temp_path=off;

server {
    location / {
        proxy_pass http://backend:3000;
        proxy_cache my_cache;
        proxy_cache_valid 200 10m;   # cache les réponses 200 pendant 10 minutes
        proxy_cache_valid 404 1m;    # cache aussi les 404, mais brièvement

        add_header X-Cache-Status $upstream_cache_status;   # HIT/MISS/EXPIRED, utile pour déboguer
    }
}
```

`X-Cache-Status` renvoyé au client permet de vérifier immédiatement si une réponse vient du cache (`HIT`) ou a été recalculée par le backend (`MISS`), sans avoir à consulter les logs.

## Load balancing : répartir le trafic entre plusieurs backends

```nginx
upstream backend_pool {
    # round-robin par défaut : chaque serveur reçoit les requêtes à tour de rôle
    server backend1.example.com:3000;
    server backend2.example.com:3000;
    server backend3.example.com:3000 backup;   # utilisé seulement si les autres sont down
}

server {
    location / {
        proxy_pass http://backend_pool;
        proxy_set_header Host $host;
    }
}
```

| Algorithme | Comportement |
|--------------|------------------|
| Round-robin (défaut) | Distribue les requêtes à tour de rôle entre les serveurs |
| `least_conn` | Envoie au serveur ayant actuellement le moins de connexions actives |
| `ip_hash` | Un même client (par IP) est toujours dirigé vers le même serveur — utile pour des sessions non partagées entre backends |

```nginx
upstream backend_pool {
    least_conn;
    server backend1:3000;
    server backend2:3000;
}
```

> [!tip] `ip_hash` est un pis-aller, pas une solution de session
> Router systématiquement un client vers le même serveur évite un problème de session non partagée, mais concentre aussi le trafic de façon inégale si les clients ne sont pas uniformément répartis. Une session partagée entre backends (Redis, base de données) reste préférable à long terme à `ip_hash`.

## Cas particuliers

> [!warning] Le cache proxy peut servir du contenu périmé après une mise à jour backend
> `proxy_cache_valid 200 10m` signifie qu'une réponse peut rester servie depuis le cache jusqu'à 10 minutes après un changement côté backend. Pour un contenu qui doit toujours être à jour, réduire `proxy_cache_valid` ou exclure certaines routes du cache plutôt que d'appliquer un cache global.

> [!info] Compression et cache se combinent, mais dans cet ordre
> Nginx compresse généralement la réponse après l'avoir servie depuis le cache (ou depuis le backend) — les deux mécanismes sont indépendants et peuvent être activés simultanément sans conflit particulier.
