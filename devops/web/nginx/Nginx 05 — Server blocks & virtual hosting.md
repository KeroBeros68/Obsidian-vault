#devops #nginx #virtual-hosting

## Un `server` = un site

```nginx
server {
    listen __NGINX_PORT__ ssl;
    listen [::]:__NGINX_PORT__ ssl;

    server_name __DOMAIN_NAME__;

    absolute_redirect off;

    root /var/www/html;
    index index.php index.html index.htm;
}
```

Un bloc `server` représente un site (une adresse virtuelle). Plusieurs blocs `server` peuvent cohabiter dans un même fichier ou la même instance Nginx — c'est le principe du **virtual hosting**.

## Cas courants

| Situation | Syntaxe / Approche |
|-----------|-------------------|
| Écouter en IPv4 et IPv6 | `listen 443 ssl;` + `listen [::]:443 ssl;` |
| Distinguer plusieurs sites sur le même port | `server_name` différent par bloc (utilise le SNI en TLS, le header `Host` en HTTP) |
| Dossier racine des fichiers servis | `root /var/www/html;` |
| Fichier servi par défaut si l'URL pointe sur un dossier | `index index.php index.html index.htm;` (testés dans l'ordre) |
| Le service est derrière un proxy externe qui change le port visible du client | `absolute_redirect off;` |

## Comment Nginx choisit le bon `server`

1. Filtre d'abord par `listen` (l'IP:port de la requête doit matcher un `listen`).
2. Parmi les blocs restants, cherche une correspondance `server_name` exacte, puis un wildcard en début (`*.example.com`), puis un wildcard en fin (`www.example.*`), puis une regex, puis se rabat sur le `server` marqué `default_server` (ou le premier bloc défini, en son absence).

## Cas particuliers

> [!warning] `absolute_redirect off` — utile derrière un reverse proxy externe
> Sans ce réglage, une redirection interne de Nginx (ex. `/wp-admin` → `/wp-admin/`) régénère une URL absolue avec le port interne du conteneur, potentiellement invisible ou incorrect côté client si un autre reverse proxy fait déjà la terminaison sur un port différent. `absolute_redirect off` conserve une redirection relative, qui reste valide quel que soit le port externe.

> [!tip] `index` teste ses arguments dans l'ordre
> `index index.php index.html index.htm;` signifie : essayer `index.php` en premier, puis `index.html`, puis `index.htm` — le premier fichier existant dans le dossier est utilisé.
