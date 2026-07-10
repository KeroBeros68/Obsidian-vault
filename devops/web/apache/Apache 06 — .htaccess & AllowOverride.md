#devops #apache #configuration #intermédiaire

## AllowOverride : autoriser (ou non) un .htaccess local

Un fichier `.htaccess` déposé dans un dossier servi par Apache peut surcharger localement la configuration — mais seulement si `AllowOverride` l'y autorise explicitement dans le `<Directory>` correspondant.

```apache
<Directory /var/www/monsite>
    AllowOverride None   # .htaccess ignorés dans ce dossier
</Directory>
```

```apache
<Directory /var/www/wordpress>
    AllowOverride All    # .htaccess pleinement pris en compte
</Directory>
```

## Pourquoi AllowOverride None est recommandé en production

- **Performance** : avec `None`, Apache ne cherche même pas de `.htaccess` à chaque requête ; avec `All`, il vérifie ce fichier (et ceux de tous les dossiers parents) à chaque accès.
- **Gouvernance** : toute la configuration reste centralisée dans les fichiers de `VirtualHost`, pas dispersée dans des `.htaccess` répartis sur le disque.
- **Sécurité** : un `.htaccess` modifiable (par un utilisateur FTP, un plugin CMS compromis) ne peut pas altérer silencieusement le comportement du serveur.

> [!tip] N'activer AllowOverride All que là où c'est réellement nécessaire
> Une application qui exige `.htaccess` (WordPress et son système de permaliens, notamment) justifie `AllowOverride All`, mais uniquement pour le `<Directory>` de cette application précise — pas globalement sur `/var/www/` ou `/`.

## Cas d'usage typique : .htaccess pour la réécriture d'URL

```apache
# .htaccess dans le dossier WordPress
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
```

Ce pattern nécessite `mod_rewrite` activé (voir [[Apache 03 — Modules (a2enmod)]]) **et** `AllowOverride All` (ou au minimum `AllowOverride FileInfo`) sur le dossier concerné — sans les deux, les règles de réécriture du `.htaccess` sont silencieusement ignorées.

## Cas particuliers

> [!warning] 500 Internal Server Error causé par un .htaccess
> Une syntaxe invalide dans un `.htaccess` (ou une directive nécessitant un module non chargé) provoque une erreur 500 — toujours vérifier `error.log` en premier (voir [[Apache 13 — Dépannage]]) avant de chercher ailleurs dans l'application.

> [!info] AllowOverride ne s'applique qu'aux dossiers, pas aux VirtualHost entiers
> `AllowOverride` se déclare dans un bloc `<Directory>`, jamais directement au niveau d'un `<VirtualHost>` — un même VirtualHost peut avoir plusieurs `<Directory>` avec des réglages `AllowOverride` différents selon les sous-dossiers.
