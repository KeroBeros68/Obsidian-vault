#devops #apache #pièges #erreurs #debugging

## 🪤 Piège 1 — `mod_php` avec le MPM `event`

```bash
# ❌ mod_php n'est pas thread-safe : crashs aléatoires possibles
apache2ctl -M | grep php   # php_module présent + MPM event actif = combinaison à risque
```

> [!warning] Rester sur `prefork` avec mod_php
> Voir [[Apache 02 — Architecture & MPM]] — passer par PHP-FPM (`proxy_fcgi`) lève cette contrainte et permet d'utiliser `event`.

---

## 🪤 Piège 2 — `AllowOverride All` partout par facilité

```apache
<!-- ❌ Performance dégradée (recherche de .htaccess à chaque requête) + gouvernance éclatée -->
<Directory /var/www/>
    AllowOverride All
</Directory>
```

```apache
<!-- ✅ None par défaut, All seulement là où c'est réellement nécessaire -->
<Directory /var/www/wordpress>
    AllowOverride All
</Directory>
```

> [!tip] Mémo
> `AllowOverride None` en production, exception ciblée par dossier si une application l'exige — voir [[Apache 06 — .htaccess & AllowOverride]].

---

## 🪤 Piège 3 — Oublier `a2enmod ssl` avant Certbot

```bash
# ❌ Certbot échoue silencieusement à configurer le VirtualHost HTTPS
sudo certbot --apache -d exemple.com
```

> [!warning] L'ordre compte
> Activer le module (`sudo a2enmod ssl`) avant de lancer Certbot — voir [[Apache 10 — Certificats Let's Encrypt avec Certbot]].

---

## 🪤 Piège 4 — Permissions 777 pour "faire passer" une erreur 403

```bash
# ❌ Résout le symptôme, ouvre une faille de sécurité majeure
sudo chmod -R 777 /var/www/monsite
```

```bash
# ✅ Permissions correctes : lecture pour le user Apache, pas d'écriture globale
sudo chown -R www-data:www-data /var/www/monsite
sudo chmod -R 755 /var/www/monsite
```

> [!warning] 777 n'est jamais la solution
> Voir [[Apache 09 — Sécurité de base]] et [[Apache 13 — Dépannage]] pour la démarche correcte face à un 403.

---

## 🪤 Piège 5 — Confondre Debian et RHEL

```bash
# ❌ a2ensite n'existe pas sur RHEL/Rocky
sudo a2ensite monsite.conf
```

> [!tip] Vérifier la distribution avant de suivre un tutoriel
> `cat /etc/os-release | grep -E '^(NAME|ID)='` — voir [[Apache 00 — Installation]] et [[Apache 04 — Structure du fichier de configuration]] pour les différences complètes.

---

## 🪤 Piège 6 — SELinux non configuré pour le reverse proxy

```
❌ 502 Bad Gateway mystérieux sur RHEL/Rocky, alors que le backend répond en direct
```

```bash
# ✅
sudo setsebool -P httpd_can_network_connect 1
```

> [!warning] SELinux, pas la configuration Apache, est souvent la cause
> Voir [[Apache 07 — Reverse proxy]] et [[Apache 13 — Dépannage]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `mod_php` + MPM `event` | Rester sur `prefork`, ou migrer vers PHP-FPM |
| `AllowOverride All` généralisé | `None` par défaut, exception ciblée par dossier |
| Certbot lancé sans `mod_ssl` activé | `a2enmod ssl` avant `certbot --apache` |
| Permissions 777 pour résoudre un 403 | `chown` + `chmod 755`/`644` |
| Commandes Debian utilisées sur RHEL (ou l'inverse) | Vérifier `/etc/os-release` avant de suivre un tutoriel |
| 502 sur reverse proxy RHEL/Rocky | `setsebool -P httpd_can_network_connect 1` |
