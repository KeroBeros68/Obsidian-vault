#devops #caddy #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Caddyfile** | Fichier de configuration de Caddy, structuré en site blocks — équivalent de `nginx.conf` ou d'un fichier `<VirtualHost>` Apache. |
| **Site block** | Bloc `domaine { }` définissant l'adresse écoutée, premier niveau du Caddyfile. |
| **Matcher** | Filtre sur une requête (chemin, méthode, en-tête), nommé (`@websocket`) ou inline (`/api/*`). |
| **Handler** | Directive qui traite effectivement la requête (`file_server`, `reverse_proxy`, `respond`...). |
| **HTTPS automatique** | Obtention et renouvellement de certificat Let's Encrypt sans configuration TLS explicite, dès qu'un domaine public est déclaré. |
| **`tls internal`** | Mode TLS utilisant la CA interne de Caddy pour générer un certificat auto-signé, pour un usage LAN/développement. |
| **`caddy trust`** | Commande installant la CA interne de Caddy dans le trust store système, pour éviter les avertissements sur les certificats `tls internal`. |
| **DNS Challenge** | Méthode de validation de domaine par Let's Encrypt via un enregistrement DNS, utilisée quand les ports 80/443 ne sont pas accessibles ou pour un certificat wildcard. |
| **`handle_path`** | Handler retirant le préfixe de chemin avant de transmettre au backend — équivalent du slash final dans `proxy_pass` chez Nginx. |
| **`lb_policy`** | Algorithme de répartition de charge d'un `reverse_proxy` multi-backend (`round_robin`, `least_conn`...). |
| **`health_uri` / `health_interval`** | Vérification active périodique de la santé de chaque backend d'un pool de reverse proxy. |
| **`basic_auth`** | Directive d'authentification HTTP Basic (remplace `basicauth`, dépréciée depuis Caddy v2.8). |
| **`caddy hash-password`** | Commande générant le hash bcrypt attendu par `basic_auth`. |
| **`xcaddy`** | Outil de build permettant de compiler Caddy avec des plugins additionnels (ex. fournisseurs DNS pour le DNS Challenge). |
| **API d'administration (port 2019)** | Port exposant les métriques Prometheus et l'API de configuration dynamique de Caddy — à ne jamais exposer publiquement. |
