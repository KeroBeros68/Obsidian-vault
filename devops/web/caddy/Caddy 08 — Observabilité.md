#devops #caddy #monitoring #avancé

## Logs structurés

```caddyfile
example.com {
    log {
        output file /var/log/caddy/access.log {
            roll_size 100mb
            roll_keep 5
        }
        format json
    }
    file_server
}
```

`format json` produit des logs structurés directement exploitables par un agrégateur (Elasticsearch, Loki...), sans parsing supplémentaire — contrairement au format texte par défaut de `access.log` chez Nginx/Apache.

`roll_size`/`roll_keep` gèrent la rotation directement dans Caddy, sans dépendre de `logrotate` côté système (voir [[Serveurs Web — Checklist production & dépannage]] pour la vérification de rotation chez Nginx/Apache).

## Métriques Prometheus

```caddyfile
{
    servers {
        metrics
    }
}

example.com {
    file_server
}
```

Le premier bloc (sans domaine, au niveau global) active les métriques ; elles sont exposées sur `:2019/metrics` par défaut.

```bash
curl localhost:2019/metrics
```

> [!warning] Le port 2019 est celui de l'API d'administration Caddy
> Ce port sert aussi à l'API de configuration dynamique de Caddy — le restreindre à `127.0.0.1` ou à un réseau interne, jamais l'exposer publiquement, pour les mêmes raisons que l'endpoint `/nginx_status` chez Nginx (voir [[Nginx 15 — Monitoring]]) ou `/server-status` chez Apache (voir [[Apache 12 — Monitoring]]).

## Cas particuliers

> [!info] Pas d'équivalent direct à stub_status/mod_status
> Caddy n'a pas de page de statut minimale équivalente à `stub_status` (Nginx) ou `server-status` (Apache) — les métriques Prometheus remplissent ce rôle directement, sans étape intermédiaire de type "aperçu rapide sans outil externe".

> [!tip] `journalctl` reste la première source de logs du service lui-même
> Les logs applicatifs (`access.log`) documentent le trafic ; `journalctl -u caddy -f` (voir [[Caddy 09 — Dépannage]]) documente le comportement du processus Caddy lui-même — les deux sources sont complémentaires, pas interchangeables.
