#devops #apache #monitoring #avancé

## Le module mod_status intégré

Équivalent Apache du `stub_status` de Nginx (voir [[Nginx 15 — Monitoring]]) : un endpoint minimal exposant l'activité courante du serveur, sans outil externe.

```apache
<IfModule mod_status.c>
    ExtendedStatus On
    <Location /server-status>
        SetHandler server-status
        Require ip 127.0.0.1
    </Location>
</IfModule>
```

```bash
curl http://127.0.0.1/server-status
```

| Directive | Rôle |
|-----------|------|
| `SetHandler server-status` | Active la page de statut sur ce chemin précis |
| `ExtendedStatus On` | Ajoute le détail par requête/thread (URL en cours, temps de traitement) — sans cette directive, la page reste sommaire |
| `Require ip 127.0.0.1` | Restreint l'accès à la machine locale (syntaxe Apache 2.4 ; l'ancienne syntaxe `Order Deny,Allow` / `Allow from` reste vue dans d'anciennes configurations mais est dépréciée) |

> [!warning] Ne jamais exposer `/server-status` publiquement
> Cette page révèle l'activité détaillée du serveur (requêtes en cours, IP clientes) sans authentification propre — la restreindre par IP (comme ci-dessus) ou par un accès authentifié, jamais laissée accessible depuis Internet.

## Activer le module

```bash
# Debian/Ubuntu
sudo a2enmod status
sudo systemctl restart apache2
```

```bash
# RHEL/Rocky : généralement chargé par défaut
httpd -M | grep status
```

## Monitoring durable : Prometheus

Pour un monitoring qui dépasse un `curl` ponctuel, un exportateur (ex. `apache_exporter` de la communauté Prometheus) scrape `/server-status?auto` (format texte simplifié, pensé pour être parsé) et expose des métriques Prometheus — même logique que `nginx-prometheus-exporter` côté Nginx (voir [[Nginx 15 — Monitoring]]), avec un exportateur différent puisque le format de sortie de `mod_status` n'est pas celui de `stub_status`.

## Cas particuliers

> [!info] `?auto` pour un format exploitable par un script
> `curl http://127.0.0.1/server-status?auto` retourne une version texte brute, clé-valeur, pensée pour être parsée automatiquement — la page HTML par défaut (sans `?auto`) est pensée pour une lecture humaine dans un navigateur.

> [!tip] ExtendedStatus a un coût marginal
> Activer `ExtendedStatus On` ajoute un léger overhead (Apache doit maintenir l'état détaillé de chaque requête en cours) — négligeable pour la plupart des déploiements, mais à garder en tête sur un serveur avec un volume de requêtes extrêmement élevé.
