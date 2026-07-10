#devops #apache #reverse-proxy #intermédiaire

## Activer les modules proxy

```bash
# Debian/Ubuntu
sudo a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests
sudo systemctl restart apache2
```

```bash
# RHEL/Rocky : généralement déjà activés, vérifier avec :
httpd -M | grep proxy
```

## Configuration de base

```apache
<VirtualHost *:80>
    ServerName api.exemple.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"

    ErrorLog ${APACHE_LOG_DIR}/api-error.log
    CustomLog ${APACHE_LOG_DIR}/api-access.log combined
</VirtualHost>
```

| Directive | Rôle |
|-----------|------|
| `ProxyPass` | Transmet les requêtes reçues vers le backend indiqué — équivalent de `proxy_pass` chez Nginx (voir [[Nginx 10 — Reverse proxy HTTP & WebSockets]]) |
| `ProxyPassReverse` | Réécrit les en-têtes de redirection renvoyés par le backend, pour qu'ils pointent vers l'URL publique et non vers l'adresse interne |
| `ProxyPreserveHost On` | Transmet le `Host:` original du client au backend, plutôt que celui du backend lui-même |
| `RequestHeader set` | Ajoute des en-têtes personnalisés transmis au backend (IP réelle, protocole d'origine) |

## Reverse proxy sur un chemin spécifique

```apache
<VirtualHost *:80>
    ServerName exemple.com
    DocumentRoot /var/www/frontend

    # L'API sur /api/*, le reste servi en statique
    ProxyPass /api/ http://127.0.0.1:3000/
    ProxyPassReverse /api/ http://127.0.0.1:3000/

    <Directory /var/www/frontend>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

## Load balancing avec mod_proxy_balancer

Répartir le trafic entre plusieurs instances backend, avec bascule automatique en cas de panne :

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://127.0.0.1:3001"
    BalancerMember "http://127.0.0.1:3002"
    BalancerMember "http://127.0.0.1:3003" status=+H   # +H = hot standby, utilisé si les autres tombent
    ProxySet lbmethod=byrequests   # round-robin par défaut ; alternative : bytraffic, bybusyness
</Proxy>

ProxyPass "/" "balancer://mycluster/"
ProxyPassReverse "/" "balancer://mycluster/"
```

| Algorithme (`lbmethod`) | Comportement |
|---------------------------|------------------|
| `byrequests` (défaut) | Round-robin par nombre de requêtes |
| `bytraffic` | Répartit selon le volume d'octets transmis |
| `bybusyness` | Envoie au membre le moins occupé (requêtes en cours) |

> [!info] Équivalent de l'`upstream` Nginx
> Ce bloc `<Proxy balancer://...>` joue le même rôle que l'`upstream` Nginx (voir [[Nginx 13 — Cache, compression & load balancing]]) — la syntaxe diffère, le principe (plusieurs backends, un algorithme de répartition, un membre de secours) reste identique.

## Cas particuliers

> [!warning] SELinux bloque le proxy sur RHEL/Rocky
> Un `502 Bad Gateway` alors que le backend répond correctement en direct signale souvent SELinux : `sudo setsebool -P httpd_can_network_connect 1` autorise Apache à initier des connexions réseau sortantes.

> [!tip] ProxyPreserveHost, souvent oublié
> Sans `ProxyPreserveHost On`, le backend reçoit son propre nom d'hôte interne (`127.0.0.1` ou similaire) au lieu du domaine public demandé par le client — une application qui génère des URLs absolues à partir de ce header peut produire des liens incorrects.
