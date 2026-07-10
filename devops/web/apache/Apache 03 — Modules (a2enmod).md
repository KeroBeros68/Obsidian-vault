#devops #apache #configuration #fondamentaux

## Apache est modulaire par défaut

Pour économiser la mémoire, Apache ne charge que le strict minimum au démarrage — chaque fonctionnalité additionnelle (TLS, réécriture d'URL, reverse proxy...) est un **module** à activer explicitement.

| Module | Ce qu'il fait | Quand l'activer |
|--------|------------------|----------------------|
| `ssl` | HTTPS | Dès qu'un chiffrement TLS est nécessaire — voir [[Apache 08 — TLS-SSL]] |
| `rewrite` | Réécriture d'URL | WordPress, redirections, URLs propres |
| `proxy` (+ `proxy_http`) | Reverse proxy | Apache devant une application Node/Python — voir [[Apache 07 — Reverse proxy]] |
| `headers` | Modifier les en-têtes HTTP | Sécurité, cache, CORS |
| `deflate` | Compression gzip | Quasiment toujours recommandé — voir [[Apache 11 — Performance]] |
| `status` | Page de diagnostic `/server-status` | Monitoring — voir [[Apache 12 — Monitoring]] |

## Activer/désactiver un module (Debian/Ubuntu)

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2

sudo a2dismod rewrite
sudo systemctl restart apache2
```

`a2enmod`/`a2dismod` créent ou suppriment un lien symbolique entre `mods-available/` et `mods-enabled/` — ce sont des scripts fournis par le paquet Debian, pas une fonctionnalité d'Apache lui-même.

## Sur RHEL/Rocky : pas d'équivalent à a2enmod

```bash
# Les modules sont généralement déjà activés par défaut ; vérifier avec :
httpd -M | grep proxy
```

Sur RHEL, l'activation d'un module se fait en décommentant sa ligne `LoadModule` directement dans les fichiers de `/etc/httpd/conf.modules.d/` — pas de script dédié.

## Lister les modules actifs

```bash
apache2ctl -M   # Debian/Ubuntu
httpd -M        # RHEL/Rocky
```

## Cas particuliers

> [!warning] Un module oublié bloque silencieusement une fonctionnalité
> Une configuration qui référence `SSLEngine On` sans que `mod_ssl` soit chargé, ou `ProxyPass` sans `mod_proxy`, échoue au chargement de la configuration (`Invalid command`) — toujours vérifier la présence du module avant de chercher une erreur ailleurs dans la syntaxe.

> [!tip] Activer un module avant de lancer Certbot
> Oublier `a2enmod ssl` avant d'exécuter Certbot (voir [[Apache 10 — Certificats Let's Encrypt avec Certbot]]) est une cause fréquente d'échec silencieux de l'activation HTTPS.
