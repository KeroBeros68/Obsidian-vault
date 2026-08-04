#bdd #adminer #avancé #déploiement #nginx #docker

## Déploiement classique : nginx + PHP-FPM

Adminer est un script PHP comme un autre du point de vue du serveur web — même modèle que n'importe quelle application PHP servie derrière nginx.

```nginx
server {
    listen 443 ssl;
    server_name db-admin.interne.example.com;

    root /var/www/adminer;
    index admin-db-x7f2k9.php;   # nom renommé, voir [[Adminer 03 — Sécurisation]]

    location ~ \.php$ {
        allow 10.0.0.0/8;
        deny all;

        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

> [!tip] Pool PHP-FPM dédié
> Faire tourner Adminer dans un pool PHP-FPM séparé de celui de l'application principale (utilisateur système distinct, `pm = ondemand`) limite l'impact d'une éventuelle compromission — l'isolation par pool est une pratique standard pour tout script d'administration sensible, pas spécifique à Adminer.

## Déploiement via Docker

L'image officielle embarque son propre serveur PHP intégré, écoutant sur le port `8080` — pas besoin de nginx/PHP-FPM séparé pour un usage ponctuel :

```yaml
services:
  adminer:
    image: adminer:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:8080"   # n'exposer que sur localhost, jamais 0.0.0.0 sans réflexion
    environment:
      ADMINER_DEFAULT_SERVER: mysql        # pré-remplit le champ serveur
      ADMINER_DESIGN: nette                # thème d'interface
    networks:
      - backend

networks:
  backend:
    internal: true   # pas de sortie réseau directe depuis ce réseau
```

| Variable d'environnement | Rôle |
|---------------------------|------|
| `ADMINER_DEFAULT_SERVER` | Pré-remplit le champ hôte du formulaire de connexion (ex. nom du service `db` dans le même compose) |
| `ADMINER_DESIGN` | Sélectionne un thème parmi ceux embarqués dans l'image |
| `ADMINER_PLUGINS` | Charge des plugins listés, à combiner avec des scripts déposés dans `/var/www/html/plugins-enabled/` |

> [!warning] `internal: true` isole le réseau Docker, pas l'accès au port publié
> Le réseau interne empêche les conteneurs d'atteindre Internet, mais la ligne `ports:` reste ce qui détermine l'exposition externe. Publier sur `127.0.0.1:8080` plutôt que `8080` seul évite l'exposition à toutes les interfaces réseau de l'hôte.

## Déploiement derrière un reverse proxy avec authentification

Pattern recommandé pour un accès occasionnel par une équipe restreinte : Adminer reste injoignable directement, un reverse proxy avec authentification (OAuth2 Proxy, Authelia, ou simple Basic Auth) filtre l'accès en amont.

```
Internet ──✗── Adminer (jamais exposé directement)
Internet ──✓── Reverse proxy (TLS + auth) ──── réseau interne ──── Adminer ──── base de données
```

Cette architecture déplace la charge de sécurité sur un composant dédié à l'authentification, plutôt que de compter sur les seules protections intégrées à Adminer.

## Pour aller plus loin

La liste des mesures de sécurité elles-mêmes (restriction IP, renommage, mise à jour) est détaillée dans [[Adminer 03 — Sécurisation]]. Pour charger des plugins personnalisés au démarrage du conteneur, voir [[Adminer 05 — Plugins & personnalisation]].

Sources : [Adminer — Docker Hub (image officielle)](https://hub.docker.com/_/adminer/), [docker-library/docs — adminer](https://github.com/docker-library/docs/tree/master/adminer), [Adminer — site officiel](https://www.adminer.org/en/)
