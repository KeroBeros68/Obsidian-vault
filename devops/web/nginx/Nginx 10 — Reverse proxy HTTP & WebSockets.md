#devops #nginx #reverse-proxy #intermédiaire

## proxy_pass vers un backend HTTP

[[Nginx 07 — Reverse proxy vers un backend (FastCGI)]] couvre le cas PHP-FPM (protocole FastCGI). Pour un backend qui parle HTTP directement (Node.js, Python, Go...), c'est `proxy_pass` qu'il faut utiliser — pas `fastcgi_pass` (voir [[Nginx — Pièges classiques]]).

```nginx
location / {
    proxy_pass http://127.0.0.1:3000;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

| Header | Rôle |
|--------|------|
| `Host` | Transmet le domaine d'origine — sans lui, l'application peut générer des URLs incorrectes |
| `X-Real-IP` | IP réelle du client (sinon le backend ne voit que l'IP de Nginx) |
| `X-Forwarded-For` | Chaîne des IP traversées, utile si plusieurs proxys sont en chaîne |
| `X-Forwarded-Proto` | Indique si la requête d'origine était en HTTP ou HTTPS |

## Le slash final dans proxy_pass : un piège classique

```nginx
# Sans slash final sur l'URL du backend : l'URI complète est transmise telle quelle
location /api/ {
    proxy_pass http://backend:3000;
}
# /api/users → http://backend:3000/api/users

# Avec slash final : le préfixe /api/ est retiré de l'URI transmise
location /api/ {
    proxy_pass http://backend:3000/;
}
# /api/users → http://backend:3000/users
```

> [!warning] Un seul caractère change tout le comportement
> La présence ou l'absence du `/` final après l'adresse du backend dans `proxy_pass` détermine si le préfixe de la `location` est conservé ou retiré de l'URI transmise. En cas de doute, tester avec `curl -v` plutôt que de deviner.

## Timeouts pour les backends lents

```nginx
location / {
    proxy_pass http://backend:3000;
    proxy_connect_timeout 120s;   # défaut : 60s
    proxy_send_timeout 120s;
    proxy_read_timeout 120s;
    proxy_set_header Host $host;
}
```

Un traitement lourd côté backend (génération de rapport, appel à un modèle IA...) qui dépasse les timeouts par défaut se traduit par un `504 Gateway Timeout` côté client — voir [[Serveurs Web — Checklist production & dépannage]].

## WebSockets

Une connexion WebSocket démarre comme une requête HTTP classique, puis « upgrade » vers un protocole différent — Nginx doit relayer explicitement ce changement.

```nginx
location /ws/ {
    proxy_pass http://backend:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

Sans `proxy_http_version 1.1` (HTTP/1.0 ne supporte pas l'upgrade) ni les deux `proxy_set_header` dédiés, la connexion WebSocket échoue silencieusement ou retombe en HTTP classique.

## Docker/Kubernetes : forcer la résolution DNS dynamique

Par défaut, Nginx résout le nom du backend **une seule fois**, au démarrage. En environnement conteneurisé, l'IP associée à un nom de service peut changer après un redéploiement — Nginx continue alors de router vers l'ancienne IP.

```nginx
# Docker : forcer une résolution périodique via le DNS interne (voir Nginx 06 / Docker 06)
resolver 127.0.0.11 valid=10s;
set $backend "http://backend:3000";
proxy_pass $backend;
```

```nginx
# Kubernetes : utiliser le resolver du cluster
resolver kube-dns.kube-system.svc.cluster.local valid=5s;
```

> [!warning] `proxy_pass` avec une variable, pas une adresse littérale
> La résolution dynamique ne fonctionne que si `proxy_pass` reçoit une **variable** (`$backend`) plutôt qu'une adresse écrite en dur — c'est ce détour par `set` qui force Nginx à réévaluer la résolution DNS selon `valid=`, au lieu de la mettre en cache indéfiniment.

## Cas particuliers

> [!info] SELinux et le reverse proxy (RHEL/Rocky)
> Si `proxy_pass` échoue avec un `502` sur un hôte RHEL/Rocky avec SELinux actif, `sudo setsebool -P httpd_can_network_connect 1` autorise Nginx à initier des connexions réseau sortantes — un blocage fréquent, indépendant de la configuration Nginx elle-même.
