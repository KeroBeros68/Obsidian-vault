#devops #caddy #configuration #fondamentaux

## Quatre couches, dans cet ordre

Chaque requête traverse ces couches dans l'ordre — les comprendre résout la majorité des configurations Caddy.

| Couche | Rôle | Équivalent |
|--------|------|----------------|
| **Site block** | Définit le domaine ou l'adresse qui écoute | `server {}` chez Nginx, `<VirtualHost>` chez Apache |
| **Matchers** | Filtrent les requêtes par chemin, méthode, en-tête | `location` chez Nginx |
| **Handlers** | Traitent la requête (servir un fichier, proxyfier...) | Directives internes à une `location`/un `<Directory>` |
| **TLS** | Activé automatiquement pour tout domaine public | Configuration manuelle chez Nginx/Apache |

```
# Structure de base
site_block {
    directive1
    directive2
}
```

## Exemple minimal commenté

```caddyfile
example.com {
    root * /var/www/site
    file_server
}
```

- `example.com { }` : le site block — écoute sur ce domaine, ports 80/443.
- `root * /var/www/site` : définit la racine des fichiers pour toutes les requêtes (`*`).
- `file_server` : le handler qui sert effectivement les fichiers statiques.
- Rien concernant TLS n'apparaît : HTTPS s'active automatiquement dès que le domaine est public (voir [[Caddy 05 — HTTPS automatique]]).

## La règle d'or

Caddy active HTTPS automatiquement pour tout domaine public déclaré dans un site block — sans directive `tls` explicite. Si le DNS de `example.com` pointe vers le serveur, Caddy obtient un certificat Let's Encrypt sans qu'on le lui demande.

## Cas particuliers

> [!tip] Valider et formater avant de recharger
> ```bash
> caddy validate --config /etc/caddy/Caddyfile
> caddy fmt --overwrite /etc/caddy/Caddyfile   # réindente le fichier
> sudo systemctl reload caddy                   # sans coupure de service
> ```
> Même discipline que `nginx -t`/`apache2ctl configtest` avant un reload (voir [[Nginx 04 — Structure du fichier de configuration]], [[Apache 04 — Structure du fichier de configuration]]).

> [!info] Un matcher nommé se réutilise
> `@websocket` ou `@404` (voir [[Caddy 07 — Pages d'erreur & routing avancé]]) sont des matchers nommés, définis une fois puis référencés dans plusieurs handlers — évite de répéter la même condition de filtrage.
