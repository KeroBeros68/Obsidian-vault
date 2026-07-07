#devops #nginx #fastcgi #reverse-proxy

## Transmettre les scripts PHP à PHP-FPM

```nginx
location ~ \.php$ {
    try_files $uri =404;

    fastcgi_pass wordpress:__WP_PORT__;
    fastcgi_index index.php;

    include fastcgi_params;

    fastcgi_param HTTP_HOST $http_host;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

Nginx ne sait pas exécuter du PHP. Pour les fichiers `.php`, il transmet la requête à un processus PHP-FPM via le protocole **FastCGI** — un protocole binaire distinct de HTTP, pensé pour ce type de délégation.

## Rôle de chaque directive

| Directive | Rôle |
|-----------|------|
| `try_files $uri =404` | Vérifie que le fichier `.php` existe **avant** de le transmettre à PHP-FPM — renvoie une 404 directe sinon |
| `fastcgi_pass wordpress:9000` | Adresse (hôte:port) du processus PHP-FPM qui traitera la requête |
| `fastcgi_index index.php` | Fichier utilisé par défaut si l'URI se termine par `/` |
| `include fastcgi_params` | Charge un ensemble standard de variables FastCGI (méthode, query string, en-têtes...) |
| `fastcgi_param HTTP_HOST $http_host` | Transmet le `Host` original du client à PHP, pour que l'application génère des URLs correctes |
| `fastcgi_param SCRIPT_FILENAME ...` | Indique à PHP-FPM le chemin absolu **réel** du script à exécuter sur le disque |

## Cas particuliers

> [!warning] `try_files $uri =404` avant `fastcgi_pass` : une mesure de sécurité, pas un détail
> Sans cette vérification, Nginx transmettrait à PHP-FPM n'importe quelle URI se terminant en `.php`, y compris des chemins fabriqués par un client malveillant (ex. `/upload/image.php/malicious.jpg`). PHP-FPM traiterait alors un chemin qui n'est pas un vrai script PHP existant, ce qui a historiquement permis des exécutions de code arbitraires sur des configurations mal protégées. Vérifier l'existence du fichier avant de déléguer à PHP-FPM ferme cette porte.

> [!tip] `SCRIPT_FILENAME` doit correspondre au `root` réellement utilisé
> `$document_root$fastcgi_script_name` reconstruit le chemin absolu à partir du `root` du `server` actif. Si le `root` diffère entre Nginx et le conteneur PHP-FPM (montages de volumes différents), PHP-FPM cherchera le fichier au mauvais endroit et renverra "No such file or directory" même si le fichier existe côté Nginx.
