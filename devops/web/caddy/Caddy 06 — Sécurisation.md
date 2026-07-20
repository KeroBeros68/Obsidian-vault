#devops #caddy #sécurité #avancé

## Headers de sécurité

```caddyfile
example.com {
    header {
        Strict-Transport-Security "max-age=31536000"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
    }
    file_server
}
```

Mêmes en-têtes que ceux détaillés côté Nginx (voir [[Nginx 14 — Sécurisation avancée]]) et Apache (voir [[Apache 09 — Sécurité de base]]) — seule la syntaxe de déclaration change.

```caddyfile
header -Server   # supprime l'en-tête Server plutôt que d'en ajouter un
```

## Authentification basique

```caddyfile
example.com {
    basic_auth /admin/* {
        admin $2a$14$hash...
    }
    file_server
}
```

```bash
caddy hash-password   # invite à saisir un mot de passe, retourne le hash bcrypt correspondant
```

> [!info] `basic_auth` remplace `basicauth` depuis Caddy v2.8
> L'ancienne directive `basicauth` fonctionne encore mais émet un avertissement de dépréciation — utiliser `basic_auth` dans toute nouvelle configuration.

## Compression

```caddyfile
example.com {
    encode gzip zstd
    file_server
}
```

`encode` accepte plusieurs algorithmes ; Caddy négocie automatiquement celui supporté par le client, sans configuration de types MIME explicite contrairement à `gzip_types` chez Nginx (voir [[Nginx 13 — Cache, compression & load balancing]]).

## Cas particuliers

> [!warning] Un hash bcrypt, jamais un mot de passe en clair
> `basic_auth` n'accepte qu'un hash bcrypt généré par `caddy hash-password` — écrire un mot de passe en clair directement dans le Caddyfile ne fonctionne pas et exposerait le mot de passe dans un fichier de configuration versionnable.

> [!tip] Basic Auth protège l'accès, pas le transport
> Comme pour Nginx (voir [[Nginx 14 — Sécurisation avancée]]), Basic Auth n'a de sens qu'associée à HTTPS — actif par défaut avec Caddy dès qu'un domaine public est déclaré (voir [[Caddy 05 — HTTPS automatique]]), donc rarement un point d'attention supplémentaire ici contrairement à Nginx/Apache où HTTPS reste une étape manuelle séparée.
