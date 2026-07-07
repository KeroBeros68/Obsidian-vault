#devops #nginx #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier `try_files $uri =404` avant `fastcgi_pass`

```nginx
# ❌ Transmet n'importe quel chemin en .php à PHP-FPM, même inexistant
location ~ \.php$ {
    fastcgi_pass wordpress:9000;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}

# ✅ Vérifie que le fichier existe avant de le transmettre
location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass wordpress:9000;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

> [!warning] Impact concret
> Sans cette vérification, une URI fabriquée comme `/uploads/image.php/payload.jpg` peut être transmise telle quelle à PHP-FPM, qui l'interprète comme un chemin de script à exécuter — un des vecteurs classiques d'exécution de code arbitraire sur des configurations Nginx + PHP-FPM mal protégées.

---

## 🪤 Piège 2 — Confondre `proxy_pass` et `fastcgi_pass`

[Description du piège] `proxy_pass` transmet une requête HTTP complète à un serveur qui parle HTTP. `fastcgi_pass` utilise le protocole binaire FastCGI, pensé pour dialoguer avec un processus comme PHP-FPM, pas avec un serveur HTTP. Utiliser l'un à la place de l'autre ne fonctionne pas : ce ne sont pas deux syntaxes différentes pour la même chose, mais deux protocoles différents.

> [!tip] Mémo
> HTTP en face → `proxy_pass`. Processus FastCGI (PHP-FPM) en face → `fastcgi_pass`.

---

## 🪤 Piège 3 — Mauvaise priorité entre `location`

[Description du piège] Nginx ne suit pas l'ordre du fichier pour les blocs de préfixe : le préfixe le **plus long** l'emporte, et une regex sans `^~` sur le préfixe correspondant peut malgré tout prendre le dessus. Un bloc `location /api/ { ... }` défini avant un `location ~ \.php$ { ... }` n'empêche pas ce dernier de s'appliquer à une requête `/api/upload.php`, contrairement à une lecture "top to bottom" intuitive. Voir [[Nginx 06 — Routing avec location & try_files]].

> [!tip] Mémo
> Exact d'abord, puis préfixe le plus long (sauf `^~` qui court-circuite), puis la première regex qui matche, sinon le préfixe mémorisé.

---

## 🪤 Piège 4 — Laisser des protocoles TLS obsolètes actifs

```nginx
# ❌ TLSv1.0/1.1 sont obsolètes et vulnérables
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

# ✅ Profil "Intermediate" recommandé en 2026
ssl_protocols TLSv1.2 TLSv1.3;
```

> [!warning] Ce qui se passe concrètement
> TLSv1.0 et TLSv1.1 sont vulnérables à des attaques connues (ex. BEAST, POODLE selon la configuration) et ne sont plus recommandés par aucune référence de sécurité actuelle. Les laisser actifs "pour la compatibilité" expose sans bénéfice réel pour l'écrasante majorité des clients modernes. Voir [[Nginx 08 — TLS-SSL]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `fastcgi_pass` sans vérification d'existence du fichier | Toujours `try_files $uri =404;` avant |
| `proxy_pass` utilisé à la place de `fastcgi_pass` (ou l'inverse) | Vérifier le protocole attendu par le backend (HTTP vs FastCGI) |
| Supposer un ordre "top to bottom" des `location` | Connaître l'ordre réel : exact > préfixe le plus long (`^~` prioritaire) > regex dans l'ordre > repli préfixe |
| `ssl_protocols` incluant TLSv1/1.1 | Se limiter à `TLSv1.2 TLSv1.3` (profil Intermediate) |
