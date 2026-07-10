#devops #apache #virtual-hosting #intermédiaire

## Un Virtual Host = un site

Chaque `<VirtualHost>` associe un domaine (`ServerName`) à un dossier de fichiers (`DocumentRoot`) — l'équivalent du bloc `server {}` de Nginx (voir [[Nginx 05 — Server blocks & virtual hosting]]).

```apache
<VirtualHost *:80>
    ServerName www.boutique.com
    DocumentRoot /var/www/boutique
</VirtualHost>
```

## Comment Apache choisit le bon Virtual Host

Chaque requête HTTP transporte un en-tête `Host:` indiquant le domaine demandé :

```
GET /index.html HTTP/1.1
Host: www.site1.com
```

Apache compare cet en-tête aux directives `ServerName` et `ServerAlias` de chaque `VirtualHost` défini. **Le premier qui correspond gagne** ; si aucun ne correspond, Apache utilise le **premier Virtual Host défini** dans l'ordre de chargement — d'où l'importance de l'ordre des fichiers en cas d'absence de correspondance exacte.

## Plusieurs sites sur un serveur

```apache
# site1.conf
<VirtualHost *:80>
    ServerName www.site1.com
    ServerAlias site1.com
    DocumentRoot /var/www/site1

    <Directory /var/www/site1>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/site1-error.log
    CustomLog ${APACHE_LOG_DIR}/site1-access.log combined
</VirtualHost>
```

`ServerAlias` ajoute un ou plusieurs noms supplémentaires reconnus pour le même site (typiquement la version sans `www.`), sans dupliquer tout le bloc.

## Activer un site

```bash
# Debian/Ubuntu
sudo a2ensite monsite.conf
sudo a2dissite 000-default.conf   # désactiver le site par défaut, optionnel
sudo apache2ctl configtest && sudo systemctl reload apache2
```

```bash
# RHEL/Rocky : déposer le fichier dans conf.d/ suffit, pas d'activation séparée
sudo httpd -t && sudo systemctl reload httpd
```

## Voir quel Virtual Host répond

```bash
apache2ctl -S   # Debian/Ubuntu
httpd -S        # RHEL/Rocky
```

Liste tous les Virtual Hosts chargés et l'ordre dans lequel Apache les évalue — la première chose à vérifier si la page par défaut d'Apache s'affiche au lieu du site attendu.

## Cas particuliers

> [!warning] « It works » au lieu de votre site
> Deux causes possibles : le site n'a pas été activé (`a2ensite` oublié sur Debian/Ubuntu), ou `ServerName` ne correspond pas exactement au domaine demandé. `apache2ctl -S` (ou `httpd -S`) révèle immédiatement lequel des deux.

> [!info] `*:80` ne signifie pas "toutes les IP indistinctement pour tous les VirtualHost"
> `*:80` matche toutes les IP sur le port 80, mais le choix final du bon site se fait ensuite sur `ServerName`/`ServerAlias`, pas sur l'IP — plusieurs `VirtualHost *:80` cohabitent normalement, chacun distingué par son nom de domaine.
