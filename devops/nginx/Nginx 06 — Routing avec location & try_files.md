#devops #nginx #routing

## Sélectionner la bonne `location`

```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}
```

Nginx ne parcourt pas les blocs `location` dans l'ordre du fichier : il applique un **ordre de priorité fixe** basé sur le type de correspondance (modifier).

## Cas courants — priorité des modificateurs

| Modificateur | Rôle | Priorité |
|--------------|------|----------|
| `location = /uri` | Correspondance exacte | 1 — gagne immédiatement si elle matche |
| `location ^~ /uri` | Préfixe prioritaire | 2 — si c'est le préfixe le plus long qui matche, arrête la recherche (pas de test des regex) |
| `location /uri` (sans modificateur) | Préfixe simple | mémorisé comme candidat, mais testé après les regex |
| `location ~ regex` / `location ~* regex` | Regex sensible / insensible à la casse | 3 — testées dans l'ordre du fichier, la première qui matche gagne |
| (aucune regex ne matche) | — | 4 — repli sur le préfixe le plus long mémorisé |

## Illustration

```
Requête : /index.php?page=2
1. Match exact "=" ?         → non
2. Meilleur préfixe : "/"    → mémorisé, mais pas "^~" donc on continue
3. Regex "~ \.php$" ?        → oui, correspond → utilisée directement
```

## `try_files` : essayer, puis se rabattre

`try_files $uri $uri/ /index.php?$args;` teste chaque argument dans l'ordre, comme fichier puis comme dossier, et s'arrête au premier qui existe réellement sur le disque :

1. `$uri` — le fichier demandé existe-t-il tel quel ?
2. `$uri/` — existe-t-il comme dossier (sert alors son `index`) ?
3. `/index.php?$args` — dernier argument, toujours utilisé comme **redirection interne** si rien avant n'a matché (pas de test d'existence sur ce dernier argument).

## Cas particuliers

> [!warning] Le dernier argument de `try_files` n'est pas vérifié
> Contrairement aux arguments précédents, le dernier argument de `try_files` est utilisé sans condition — c'est le mécanisme qui permet de router toute URL inconnue vers `index.php` (pattern "front controller"), mais aussi celui qui doit être surveillé : elle n'échoue jamais silencieusement, elle route toujours quelque part.

> [!tip] Retenir l'ordre en une phrase
> "Exact d'abord, préfixe le plus long ensuite (sauf `^~` qui court-circuite), puis la première regex qui matche, sinon le préfixe mémorisé."
