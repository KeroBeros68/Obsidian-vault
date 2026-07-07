#devops #nginx #configuration

## Les contextes s'emboîtent

La configuration Nginx est organisée en **contextes** hiérarchiques : chaque bloc `{ }` en ouvre un nouveau, et une directive placée dans un contexte parent est héritée par ses enfants, sauf si elle y est redéfinie.

## Illustration

```
main (niveau fichier, hors accolades)
├── events { ... }        → tuning des connexions (worker_connections)
└── http { ... }          → tout ce qui concerne HTTP
    ├── server { ... }    → un site / une adresse virtuelle
    │   └── location { ... }  → une route à l'intérieur du site
    └── server { ... }    → un autre site (autre server_name ou autre port)
        └── location { ... }
```

```nginx
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 443 ssl;
        # ce server hérite de include/default_type définis dans http
    }
}
```

## Règles d'héritage et de portée

- Une directive définie dans `http` s'applique à tous les `server` qu'il contient, sauf si un `server` (ou une `location`) la redéfinit localement.
- Certaines directives sont valides uniquement dans un contexte précis : `worker_connections` n'existe que dans `events`, `server_name` n'existe que dans `server`.
- `include` permet de fragmenter la configuration (`mime.types`, `fastcgi_params`, `conf.d/*.conf`) plutôt que d'avoir un unique fichier monolithique.

## Cas particuliers

> [!warning] Une directive au mauvais niveau est ignorée ou provoque une erreur
> Placer `worker_connections` dans `http` au lieu de `events`, par exemple, fait échouer le test de configuration (`nginx -t`) — chaque directive appartient à un contexte précis, pas "n'importe où".

> [!tip] `nginx -t` avant tout reload
> Tester la configuration avec `nginx -t` avant `nginx -s reload` permet de détecter une erreur de syntaxe ou de contexte sans jamais interrompre le service en production.
