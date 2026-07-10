#devops #apache #configuration #fondamentaux

## Deux organisations de fichiers selon la distribution

| Élément | Debian/Ubuntu | RHEL/Rocky |
|---------|-------------------|----------------|
| Config principale | `/etc/apache2/apache2.conf` | `/etc/httpd/conf/httpd.conf` |
| Sites disponibles | `/etc/apache2/sites-available/` | — (n'existe pas) |
| Sites activés | `/etc/apache2/sites-enabled/` (liens symboliques) | `/etc/httpd/conf.d/` (fichiers directs) |
| Modules | `/etc/apache2/mods-available/` | `/etc/httpd/conf.modules.d/` |
| Logs | `/var/log/apache2/` | `/var/log/httpd/` |
| `DocumentRoot` par défaut | `/var/www/html` | `/var/www/html` |

Sur Debian/Ubuntu, un site créé dans `sites-available/` reste inactif tant qu'il n'est pas explicitement activé (`a2ensite`, voir [[Apache 03 — Modules (a2enmod)]]) — un niveau d'indirection absent sur RHEL, où tout fichier déposé dans `conf.d/` est automatiquement pris en compte.

## Blocs de configuration

```apache
<VirtualHost *:80>
    ServerName monsite.local
    DocumentRoot /var/www/monsite

    <Directory /var/www/monsite>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/monsite-error.log
    CustomLog ${APACHE_LOG_DIR}/monsite-access.log combined
</VirtualHost>
```

| Bloc | Rôle |
|------|------|
| `<VirtualHost>` | Un site — voir [[Apache 05 — Virtual Hosts]] |
| `<Directory>` | Règles s'appliquant à un chemin précis du disque (pas d'une URL, contrairement à `location` chez Nginx) |
| `Options` | Comportements activables/désactivables (`-Indexes` = pas de listing de dossier, `+FollowSymLinks` = suit les liens symboliques) |
| `AllowOverride` | Autorise ou non un `.htaccess` à surcharger la configuration de ce dossier — voir [[Apache 06 — .htaccess & AllowOverride]] |
| `Require all granted` | Autorise l'accès (syntaxe Apache 2.4 ; remplace l'ancien `Allow from all`) |

## Tester puis recharger

```bash
sudo apache2ctl configtest   # Debian/Ubuntu — "Syntax OK"
sudo httpd -t                # RHEL/Rocky
sudo systemctl reload apache2   # ou httpd
```

> [!warning] Toujours tester avant de recharger
> Un `systemctl reload` avec une erreur de syntaxe dans la configuration fait planter Apache plutôt que de conserver l'ancienne configuration active — contrairement à Nginx (`nginx -t` avant `reload`, voir [[Nginx 04 — Structure du fichier de configuration]]), la même discipline s'applique ici.

## Cas particuliers

> [!warning] `AH00558: Could not reliably determine the server's fully qualified domain name`
> Ce message au démarrage signale l'absence d'une directive `ServerName` au niveau global (hors `VirtualHost`) — ajouter `ServerName localhost` dans la configuration principale fait disparaître l'avertissement, sans affecter le fonctionnement des sites eux-mêmes.

> [!info] `${APACHE_LOG_DIR}` n'existe que sur Debian/Ubuntu
> Cette variable d'environnement est définie par le paquet Debian pour pointer vers `/var/log/apache2` ; sur RHEL/Rocky, le chemin complet (`/var/log/httpd/...`) doit être écrit explicitement dans la configuration.
