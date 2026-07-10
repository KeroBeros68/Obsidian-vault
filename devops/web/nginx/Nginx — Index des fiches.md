#devops #nginx

## Fiches disponibles

### Fondamentaux

- [[Nginx 00 — Installation]]
- [[Nginx 01 — Qu'est-ce que Nginx]]
- [[Nginx 02 — Fonctionnement interne]]
- [[Nginx 03 — Architecture & modèle de processus]]
- [[Nginx 04 — Structure du fichier de configuration]]

### Intermédiaire

- [[Nginx 05 — Server blocks & virtual hosting]]
- [[Nginx 06 — Routing avec location & try_files]]
- [[Nginx 07 — Reverse proxy vers un backend (FastCGI)]]
- [[Nginx 10 — Reverse proxy HTTP & WebSockets]]
- [[Nginx 11 — root, alias & recettes de routing]]

### Avancé

- [[Nginx 08 — TLS-SSL]]
- [[Nginx 09 — Sécurité de base]]
- [[Nginx 12 — Certificats Let's Encrypt avec Certbot]]
- [[Nginx 13 — Cache, compression & load balancing]]
- [[Nginx 14 — Sécurisation avancée]]
- [[Nginx 15 — Monitoring]]
- [[Nginx 16 — Dépannage]]

### Référence

- [[Nginx — Glossaire]]
- [[Nginx — Pièges classiques]]

## Prérequis & suite

- [[Serveurs Web — Choisir son serveur]] ← prérequis (pourquoi Nginx plutôt qu'Apache/Caddy)
- [[Serveurs Web — Concepts fondamentaux]] ← prérequis (HTTP, virtual hosts, reverse proxy, TLS, cache, logs — universels à tout serveur web)
- [[Serveurs Web — Checklist production & dépannage]] ← suite logique (mise en production, dépannage 403/404/502)
- [[Docker — Index des fiches]] ← prérequis (le cas d'usage de référence combine Nginx et un conteneur PHP-FPM/WordPress orchestrés via Docker Compose)
- [[Secrets — Index des fiches]] ← suite logique (gestion des certificats TLS et variables sensibles référencées dans la configuration Nginx)
- [[Apache — Index des fiches]] ← comparaison directe, mêmes concepts (Virtual Host, reverse proxy, TLS) avec une architecture et une syntaxe différentes
