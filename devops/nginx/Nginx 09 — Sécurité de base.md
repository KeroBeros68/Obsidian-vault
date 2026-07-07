#devops #nginx #sécurité

## Trois réflexes de durcissement

```nginx
user www-data;
```

```nginx
location ~ /\. {
    deny all;
}
```

## Cas courants

| Mesure | Effet |
|--------|-------|
| `user www-data;` | Les workers tournent sans privilèges root — un processus compromis reste limité aux droits de `www-data` |
| `location ~ /\. { deny all; }` | Bloque l'accès à tous les fichiers cachés (`.env`, `.git`, `.htaccess`...) qui ne devraient jamais être servis publiquement |
| `try_files $uri =404;` avant `fastcgi_pass` | Empêche de transmettre à PHP-FPM un chemin qui ne correspond à aucun fichier réel (voir [[Nginx 07 — Reverse proxy vers un backend (FastCGI)]]) |
| `server_tokens off;` | Directive additionnelle, absente de la configuration de référence mais couramment recommandée : masque la version de Nginx dans l'en-tête `Server`, réduisant l'information utile à un attaquant |

> [!info] Ajout hors configuration de référence
> `server_tokens off` n'apparaît pas dans le fichier de configuration fourni en base de ce module ; elle est ajoutée ici comme pratique standard de durcissement, signalée explicitement pour ne pas laisser croire qu'elle provient de la configuration d'origine.

## Cas particuliers

> [!warning] Le principe du moindre privilège ne s'arrête pas à `user`
> Faire tourner les workers en `www-data` limite les dégâts d'une compromission applicative, mais ne dispense pas de restreindre aussi les permissions du système de fichiers (`root /var/www/html` en lecture seule pour ce compte, écriture limitée aux seuls dossiers qui en ont réellement besoin comme les uploads).

> [!tip] `deny all` s'applique à toute regex qui matche
> `location ~ /\.` matche toute URI contenant un `/.` suivi d'un caractère — pas seulement `/.env` ou `/.git` à la racine, mais aussi dans n'importe quel sous-dossier, ce qui en fait une protection large avec une seule règle.
