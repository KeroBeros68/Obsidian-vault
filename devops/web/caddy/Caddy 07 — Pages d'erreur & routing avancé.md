#devops #caddy #routing #avancé

## Pages d'erreur personnalisées

```caddyfile
example.com {
    root * /var/www/html
    file_server

    handle_errors {
        @404 expression {err.status_code} == 404
        handle @404 {
            root * /var/www/errors
            rewrite * /404.html
            file_server
        }

        @5xx expression {err.status_code} >= 500
        handle @5xx {
            respond "Service temporairement indisponible" 503
        }
    }
}
```

`handle_errors` intercepte les codes d'erreur générés en amont ; les matchers `@404`/`@5xx` (basés sur une expression testant `{err.status_code}`) déterminent quelle réponse personnalisée renvoyer — équivalent d'`error_page` chez Nginx.

## Redirections

```caddyfile
old.example.com {
    redir https://new.example.com{uri} permanent
}
```

`{uri}` reprend le chemin et la query string d'origine dans la redirection — évite de perdre la route demandée lors du renvoi vers le nouveau domaine.

## Réécriture et réponse directe

| Directive | Rôle |
|-----------|------|
| `redir /old /new permanent` | Redirection HTTP 301 |
| `rewrite /old /new` | Réécriture interne, invisible pour le client (pas de redirection HTTP) |
| `respond "OK" 200` | Répond directement sans backend ni fichier, utile pour un healthcheck simple |

## Cas particuliers

> [!warning] `redir` (visible du client) vs `rewrite` (interne) : ne pas confondre
> `redir` renvoie un code 3xx et change l'URL affichée dans le navigateur ; `rewrite` change la requête traitée en interne sans que le client ne voie de changement d'URL — un choix erroné casse soit le SEO (rewrite là où une vraie redirection était attendue), soit l'expérience utilisateur (redirection visible là où une réécriture transparente suffisait).

> [!info] `handle` vs `handle_path` déjà vus en reverse proxy
> Le principe de matchers nommés (`@404`, `@5xx`) et de blocs `handle`/`handle_path` est cohérent dans tout le Caddyfile — voir [[Caddy 04 — Reverse proxy]] pour son usage avec `reverse_proxy`.
