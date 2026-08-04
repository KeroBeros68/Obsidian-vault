#bdd #adminer #avancé #sécurité

## Pourquoi Adminer est une cible privilégiée

Un fichier PHP unique, sans authentification propre au niveau serveur, donnant potentiellement accès à toutes les bases d'une instance : c'est un profil de cible idéal pour les scanners automatisés. Le nom de fichier par défaut (`adminer.php`) fait partie des chemins les plus testés par les bots qui parcourent Internet à la recherche d'instances oubliées.

> [!warning] Historique de vulnérabilités réelles, activement exploitées
> Adminer a fait l'objet de plusieurs CVE significatives : une SSRF (CVE-2021-21311, versions 4.0.0-4.7.8) référencée dans le catalogue CISA des vulnérabilités activement exploitées, une lecture de fichier arbitraire côté serveur (versions 1.12.0-4.6.2, corrigée en 4.6.3), une XSS (CVE-2021-29625) et un déni de service par confusion de type (CVE-2026-25892). Une instance non mise à jour est une porte d'entrée potentielle sur le serveur qui l'héberge, pas seulement sur la base de données.

## Les trois mesures non négociables

### 1. Ne jamais exposer Adminer directement sur Internet sans protection

Adminer intègre un rate-limiting sur les tentatives de connexion (protection anti brute-force basique), mais ce n'est pas un rempart suffisant à lui seul.

| Mesure | Mise en œuvre |
|--------|----------------|
| Restriction par IP | Bloc `allow`/`deny` (nginx) ou `Require ip` (Apache) sur le chemin du fichier |
| Authentification au niveau serveur web | HTTP Basic Auth (nginx `auth_basic`, Apache `.htpasswd`) en amont du PHP — une couche avant même le formulaire Adminer |
| Tunnel plutôt qu'exposition publique | VPN, tunnel SSH (`ssh -L 8080:localhost:8080`), ou accès via un reverse proxy authentifié uniquement |

```nginx
# nginx : restreindre l'accès à un fichier Adminer par IP + Basic Auth
location /adminer.php {
    allow 10.0.0.0/8;
    deny all;

    auth_basic "Administration";
    auth_basic_user_file /etc/nginx/.htpasswd-adminer;

    fastcgi_pass unix:/run/php/php-fpm.sock;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

### 2. Renommer le fichier

```bash
mv adminer.php admin-db-x7f2k9.php
```

Une mesure de sécurité par l'obscurité — insuffisante seule, mais élimine l'essentiel du bruit des scanners automatisés qui ciblent le nom par défaut.

### 3. Maintenir la version à jour

Vérifier `?script=version` ou l'écran de connexion pour le numéro de version installé, et suivre les annonces de sécurité sur `adminer.org`. Compte tenu de l'historique de CVE actives, une instance figée depuis plusieurs années est un risque, pas une simplicité opérationnelle.

## Bonnes pratiques complémentaires

- **Retirer le fichier après usage ponctuel** — le design mono-fichier rend cette opération triviale (`rm adminer.php`), à intégrer dans la routine après une intervention de maintenance.
- **Compte applicatif à privilèges limités** — se connecter avec un compte disposant uniquement des droits nécessaires à la tâche, pas avec `root`/`postgres`, pour limiter l'impact d'une session compromise.
- **TLS systématique** — les identifiants transitent en clair sur HTTP ; jamais d'exposition, même interne, sans HTTPS.
- **Plugin de connexion renforcée** — des plugins communautaires ajoutent un support OTP ou une authentification externe, listés sur la page Plugins d'`adminer.org` — voir [[Adminer 05 — Plugins & personnalisation]].

> [!tip] Le design mono-fichier est aussi un atout sécurité
> Contrairement à une installation phpMyAdmin persistante, Adminer se déploie et se supprime en une commande — la meilleure protection reste de ne le laisser en place que le temps strictement nécessaire.

## Pour aller plus loin

Le déploiement derrière un reverse proxy avec TLS et l'usage via Docker (accès limité au réseau interne du compose) sont détaillés dans [[Adminer 04 — Déploiement (nginx, Docker, reverse proxy)]].

Sources : [Adminer — site officiel](https://www.adminer.org/en/), [CVE-2021-21311 — Adminer SSRF (SentinelOne)](https://www.sentinelone.com/vulnerability-database/cve-2021-21311/), [CVE-2021-29625 — Adminer XSS (SentinelOne)](https://www.sentinelone.com/vulnerability-database/cve-2021-29625/), [CVE-2026-25892 — Adminer DoS (SentinelOne)](https://www.sentinelone.com/vulnerability-database/cve-2026-25892/), [CISA ICS Advisory — Adminer dans des produits industriels](https://www.cisa.gov/news-events/ics-advisories/icsa-22-130-01)
