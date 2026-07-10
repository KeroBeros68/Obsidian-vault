#devops #nginx #routing #intermédiaire

## root vs alias : le chemin est complété ou remplacé

Deux directives qui semblent proches mais se comportent différemment sur la partie de l'URI capturée par la `location`.

```nginx
# root : le chemin de la location est AJOUTÉ à la racine
location /images/ {
    root /var/www;
}
# /images/photo.jpg → /var/www/images/photo.jpg

# alias : le chemin de la location est REMPLACÉ par l'alias
location /images/ {
    alias /data/photos/;
}
# /images/photo.jpg → /data/photos/photo.jpg
```

> [!tip] Règle simple pour choisir
> `root` par défaut — le chemin sur le disque reflète naturellement la structure de l'URL. `alias` seulement pour mapper une URL vers un dossier physique complètement différent de sa structure apparente.

> [!warning] `alias` nécessite un slash final cohérent
> Un `alias` sans le slash final attendu (`/data/photos` au lieu de `/data/photos/`) produit des chemins concaténés incorrectement — vérifier systématiquement la présence du slash final des deux côtés (`location /images/` et `alias .../`).

## SPA (Single Page Application)

Une application React/Vue/Angular gère son propre routing côté client — toute route inconnue côté serveur doit retomber sur `index.html`, pas sur une 404.

```nginx
location / {
    root /var/www/app;
    try_files $uri $uri/ /index.html;
}
```

Ce pattern diffère de celui vu dans [[Nginx 06 — Routing avec location & try_files]] (`try_files $uri $uri/ =404;`) uniquement par le dernier argument : une redirection interne vers `index.html` plutôt qu'une erreur — laissant l'application JS gérer la route demandée.

## Page de maintenance

```nginx
location / {
    if (-f /var/www/maintenance.html) {
        return 503;
    }
    proxy_pass http://backend;
}

error_page 503 /maintenance.html;
location = /maintenance.html {
    root /var/www;
}
```

La présence du fichier `maintenance.html` sur le disque bascule tout le trafic vers une page 503, sans toucher à la configuration ni redémarrer Nginx — il suffit de créer ou supprimer ce fichier.

## Upload de fichiers volumineux

```nginx
location /upload/ {
    client_max_body_size 100M;
    proxy_pass http://backend;
    proxy_read_timeout 300s;
}
```

`client_max_body_size` vaut 1 Mo par défaut — un upload plus volumineux échoue avec une erreur `413 Request Entity Too Large` sans ce réglage explicite.

## Cas particuliers

> [!warning] `if` dans un bloc `location` reste fragile
> La directive `if` de Nginx a une réputation méritée d'imprévisibilité dans certains contextes (documentée par Nginx lui-même comme « evil » dans sa documentation officielle). Le cas de la page de maintenance ci-dessus reste un usage courant et sûr, mais éviter d'empiler plusieurs `if` complexes dans une même `location`.

> [!info] Plusieurs sites, un fichier par site
> Chaque site sous `/etc/nginx/conf.d/` (ou `sites-available/` sur Debian, voir [[Nginx 00 — Installation]]) reste indépendant — Nginx route selon `server_name`, pas selon le nom du fichier de configuration lui-même.
