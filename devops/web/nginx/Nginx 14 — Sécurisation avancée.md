#devops #nginx #sécurité #avancé

## Headers de sécurité

En complément de [[Nginx 09 — Sécurité de base]] (masquer la version, bloquer les fichiers cachés), des en-têtes HTTP additionnels renforcent la posture du navigateur client.

```nginx
server {
    add_header X-Frame-Options "SAMEORIGIN" always;              # empêche le clickjacking
    add_header X-Content-Type-Options "nosniff" always;          # empêche le sniffing de type MIME
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;   # voir Nginx 12
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';" always;
}
```

> [!warning] `X-XSS-Protection` est un header obsolète
> Ce header historique n'est plus supporté par les navigateurs modernes (retiré de Chrome dès 2019) — la protection réelle contre les attaques XSS passe désormais par une **Content-Security-Policy** correctement configurée, pas par ce header hérité. La CSP donnée en exemple est volontairement restrictive ; l'adapter aux besoins réels de l'application (sources de scripts externes, CDN...) avant de la déployer.

## Rate limiting

Protège contre les abus (scraping agressif, tentatives de brute-force) et les attaques DDoS légères.

```nginx
# Dans http {} : définir la zone
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://backend:3000;
    }
}
```

`rate=10r/s` fixe le débit moyen autorisé par IP ; `burst=20` tolère un pic ponctuel jusqu'à 20 requêtes en excès, traitées immédiatement grâce à `nodelay` plutôt que mises en file d'attente.

## Restreindre l'accès par IP

```nginx
location /admin/ {
    allow 192.168.1.0/24;
    allow 10.0.0.0/8;
    deny all;
    proxy_pass http://backend:3000;
}
```

Les règles `allow`/`deny` sont évaluées dans l'ordre d'écriture — la première qui correspond à l'IP du client s'applique, d'où l'importance de terminer par un `deny all` explicite plutôt que de compter sur un refus implicite.

## Limiter les méthodes HTTP autorisées

```nginx
location / {
    limit_except GET POST HEAD {
        deny all;
    }
    root /var/www/html;
}
```

Bloque des méthodes comme `PUT`, `DELETE` ou `TRACE` sur une route qui n'en a jamais besoin — réduit la surface d'attaque sans toucher à la logique applicative.

## Authentification basique (Basic Auth)

```bash
sudo apt install apache2-utils   # fournit htpasswd, malgré le nom du paquet
htpasswd -c /etc/nginx/.htpasswd admin
```

```nginx
location /admin/ {
    auth_basic "Zone Admin";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://backend;
}
```

> [!info] Basic Auth protège l'accès, pas le transport
> Les identifiants Basic Auth transitent encodés en base64 (pas chiffrés) dans l'en-tête HTTP — cette protection n'a de sens qu'associée à HTTPS (voir [[Nginx 12 — Certificats Let's Encrypt avec Certbot]]), jamais utilisée seule sur une connexion HTTP en clair.

## Cas particuliers

> [!tip] Combiner allowlist IP et Basic Auth pour une zone d'administration
> Une interface d'administration exposée publiquement gagne à cumuler les deux mécanismes : `allow`/`deny` limite déjà l'accès réseau, et Basic Auth ajoute une authentification pour les IP autorisées — deux couches indépendantes plutôt qu'une seule ligne de défense.
