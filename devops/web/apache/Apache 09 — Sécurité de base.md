#devops #apache #sécurité #avancé

## Masquer les informations de version

```apache
# /etc/apache2/conf-available/security.conf (Debian)
# /etc/httpd/conf.d/security.conf (RHEL)
ServerTokens Prod
ServerSignature Off
```

`ServerTokens Prod` réduit l'en-tête `Server` à `Apache` seul, sans version ni liste de modules ; `ServerSignature Off` retire la signature ajoutée en bas des pages d'erreur générées par Apache — équivalent de `server_tokens off;` chez Nginx (voir [[Nginx 09 — Sécurité de base]]).

## En-têtes de sécurité

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>
```

Nécessite `mod_headers` (voir [[Apache 03 — Modules (a2enmod)]]). Le détail de chaque en-tête et son équivalent Nginx sont couverts dans [[Nginx 14 — Sécurisation avancée]] — la logique de sécurité est identique, seule la syntaxe d'activation change.

## Désactiver le listing de répertoire globalement

```apache
<Directory />
    Options -Indexes
</Directory>
```

Sans fichier `index.html`/`index.php` dans un dossier, `Options +Indexes` afficherait la liste de tous les fichiers présents — à désactiver globalement, puis réactiver ponctuellement si un dossier précis doit rester navigable.

## Limiter les méthodes HTTP

```apache
<Directory /var/www/monsite>
    <LimitExcept GET POST HEAD>
        Require all denied
    </LimitExcept>
</Directory>
```

Bloque des méthodes comme `PUT`, `DELETE` ou `TRACE` sur un dossier qui n'en a jamais besoin — réduit la surface d'attaque sans toucher à l'application elle-même.

## Appliquer les changements

```bash
# Debian/Ubuntu
sudo a2enmod headers
sudo a2enconf security
sudo systemctl restart apache2
```

```bash
# RHEL/Rocky : créer le fichier, puis
sudo systemctl restart httpd
```

## Cas particuliers

> [!warning] Permissions 777 : une faille de sécurité, pas une solution de dépannage
> Face à une erreur 403 (voir [[Apache 13 — Dépannage]]), corriger avec `chmod 755` (dossiers) et `644` (fichiers) — jamais `777`, qui autorise l'écriture à tout utilisateur du système, y compris un processus compromis.

> [!info] AllowOverride reste une mesure de sécurité à part entière
> `AllowOverride None` (voir [[Apache 06 — .htaccess & AllowOverride]]) empêche un `.htaccess` déposé par un attaquant ayant obtenu un accès en écriture au disque de modifier le comportement du serveur — une couche de défense qui s'ajoute aux en-têtes et aux permissions.
