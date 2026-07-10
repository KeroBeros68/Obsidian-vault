#devops #apache #dépannage #avancé

## Diagnostic en 30 secondes

```bash
# 1. Apache tourne-t-il ?
systemctl status apache2   # ou httpd

# 2. La configuration est-elle valide ?
sudo apache2ctl configtest   # ou httpd -t

# 3. Quels modules sont actifs ?
apache2ctl -M   # ou httpd -M

# 4. Logs d'erreur récents
sudo tail -20 /var/log/apache2/error.log   # Debian
sudo tail -20 /var/log/httpd/error_log     # RHEL

# 5. Quel Virtual Host répond pour quel domaine ?
sudo apache2ctl -S   # ou httpd -S
```

Même méthode que pour Nginx (voir [[Nginx 16 — Dépannage]] et [[Serveurs Web — Checklist production & dépannage]]) : service actif ? configuration valide ? que disent les logs ? — dans cet ordre.

## Erreurs courantes

| Erreur | Cause probable | Première vérification |
|--------|--------------------|----------------------------|
| 403 Forbidden | Permissions, `Require all denied`, ou SELinux | Voir « Déboguer un 403 » ci-dessous |
| 404 Not Found | `DocumentRoot` incorrect ou fichier absent | `ls -la` sur le chemin exact configuré |
| 500 Internal Server Error | `.htaccess` invalide ou script backend cassé | `tail /var/log/apache2/error.log` |
| « It works » au lieu du site | Virtual Host non activé, ou `ServerName` incorrect | `apache2ctl -S` (voir [[Apache 05 — Virtual Hosts]]) |
| `AH00558: Could not reliably determine...` | `ServerName` global absent | Ajouter `ServerName localhost` (voir [[Apache 04 — Structure du fichier de configuration]]) |

## Déboguer un 403 pas à pas

```bash
# 1. Les permissions permettent-elles la lecture par www-data/apache ?
ls -la /var/www/monsite/

# 2. La configuration Directory autorise-t-elle l'accès ?
grep -A5 "Directory /var/www/monsite" /etc/apache2/sites-enabled/*

# 3. SELinux est-il en cause (RHEL/Rocky) ?
getenforce
# Si "Enforcing" : tester temporairement
sudo setenforce 0
# Si ça résout le problème, corriger les contextes plutôt que laisser SELinux désactivé :
sudo restorecon -Rv /var/www/monsite
```

> [!warning] `setenforce 0` est un test, pas une solution
> Désactiver SELinux temporairement confirme le diagnostic, mais le laisser désactivé en production supprime une couche de sécurité entière. La correction propre est `restorecon` (recontextualisation des fichiers) ou `setsebool` pour un cas précis (voir [[Apache 07 — Reverse proxy]] pour `httpd_can_network_connect`).

## Recharger sans interruption

```bash
sudo apache2ctl configtest && sudo systemctl reload apache2
# ou
sudo httpd -t && sudo systemctl reload httpd
```

> [!warning] `reload` ≠ `restart`
> `reload` relit la configuration sans couper les connexions en cours ; `restart` arrête puis relance le service, avec une coupure de service — ne jamais utiliser `restart` en production si `reload` suffit, même règle que pour Nginx.

## Cas particuliers

> [!info] Pour les erreurs de certificat spécifiquement
> Chaîne de certificat incomplète, mauvais SNI, expiration — ces symptômes se diagnostiquent avec `openssl s_client` plutôt qu'avec les commandes ci-dessus. Voir [[Apache 10 — Certificats Let's Encrypt avec Certbot]].
