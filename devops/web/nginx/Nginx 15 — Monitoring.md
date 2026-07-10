#devops #nginx #monitoring #avancé

## Le module stub_status intégré

Expose un aperçu minimal mais immédiat de l'activité du serveur, sans outil externe.

```nginx
server {
    location /nginx_status {
        stub_status on;
        allow 127.0.0.1;
        deny all;
    }
}
```

```bash
curl http://127.0.0.1/nginx_status
# Active connections: 42
# server accepts handled requests
#  1234567 1234567 9876543
# Reading: 0 Writing: 1 Waiting: 41
```

> [!warning] Ne jamais exposer ce endpoint publiquement
> `allow 127.0.0.1; deny all;` restreint l'accès à la machine locale — un `/nginx_status` accessible depuis Internet révèle des informations sur la charge du serveur sans authentification.

## Prometheus + nginx-exporter

Pour un monitoring durable au-delà d'un simple `curl` ponctuel, `nginx-prometheus-exporter` convertit `stub_status` en métriques Prometheus.

```bash
# Sur Linux, --network host évite le problème d'accès réseau du conteneur
docker run --network host nginx/nginx-prometheus-exporter:latest \
  -nginx.scrape-uri=http://127.0.0.1/nginx_status

# Alternative : IP explicite de l'hôte Nginx
docker run -p 9113:9113 nginx/nginx-prometheus-exporter:latest \
  -nginx.scrape-uri=http://192.168.1.10/nginx_status
```

> [!warning] `host.docker.internal` ne fonctionne pas nativement sur Linux
> Ce nom spécial n'existe que sous Docker Desktop (macOS/Windows) — sur un hôte Linux natif, utiliser `--network host` ou l'adresse IP explicite de la machine, sinon l'exporter ne peut pas joindre Nginx.

Une fois l'exporter lancé, configurer Prometheus pour scraper `localhost:9113/metrics`, puis visualiser dans Grafana.

## Cas particuliers

> [!warning] Nginx Amplify est en fin de vie
> Le service Nginx Amplify (F5) s'arrête le 31 janvier 2026 — ne pas l'utiliser pour un nouveau déploiement. Prometheus + Grafana (avec `nginx-prometheus-exporter`) est la voie recommandée pour tout nouveau monitoring.

> [!info] `stub_status` est basique par nature
> Il donne un aperçu global (connexions actives, requêtes traitées) mais aucune métrique par route, code de statut ou temps de réponse — pour ce niveau de détail, les logs `access.log` restés structurés (voir [[Serveurs Web — Concepts fondamentaux]]) et un outil d'analyse de logs restent nécessaires en complément.
