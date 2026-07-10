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

## 🪤 Piège 5 — Se tromper sur le slash final dans `proxy_pass`

```nginx
# ❌ Le préfixe /api/ est conservé : /api/users → http://backend:3000/api/users
location /api/ {
    proxy_pass http://backend:3000;
}
```

```nginx
# ✅ Si le backend attend /users sans préfixe : ajouter le slash final
location /api/ {
    proxy_pass http://backend:3000/;
}
```

> [!warning] Un seul caractère change le chemin transmis au backend
> Voir [[Nginx 10 — Reverse proxy HTTP & WebSockets]] pour le détail complet ; en cas de doute, `curl -v` révèle immédiatement l'URI réellement reçue par le backend.

---

## 🪤 Piège 6 — Confondre `root` et `alias`

```nginx
# ❌ Avec root, le chemin de la location s'AJOUTE : /images/photo.jpg → /var/www/images/photo.jpg (peut ne pas exister)
location /images/ {
    root /var/www;
}
```

```nginx
# ✅ Avec alias, le chemin de la location est REMPLACÉ : /images/photo.jpg → /data/photos/photo.jpg
location /images/ {
    alias /data/photos/;
}
```

> [!warning] 404 alors que le fichier existe bien sur le disque
> Confondre les deux directives fait chercher Nginx au mauvais endroit exact, sans message d'erreur explicite. Voir [[Nginx 11 — root, alias & recettes de routing]].

---

## 🪤 Piège 7 — DNS figé en environnement Docker/Kubernetes

```nginx
# ❌ Résolu une seule fois au démarrage : Nginx garde l'ancienne IP après un redéploiement du backend
proxy_pass http://backend:3000;
```

```nginx
# ✅ Force une résolution périodique via une variable
resolver 127.0.0.11 valid=10s;
set $backend "http://backend:3000";
proxy_pass $backend;
```

> [!warning] Symptôme typique : 502 uniquement après un redéploiement
> Le service redémarre avec une nouvelle IP, mais Nginx continue à router vers l'ancienne jusqu'à son propre redémarrage — voir [[Nginx 10 — Reverse proxy HTTP & WebSockets]].

---

## 🪤 Piège 8 — `try_files` sans fallback adapté à une SPA

```nginx
# ❌ Toute route gérée côté client (React Router, etc.) renvoie une 404 serveur
location / {
    try_files $uri $uri/ =404;
}
```

```nginx
# ✅ Repli vers index.html, laissant l'application JS gérer la route
location / {
    try_files $uri $uri/ /index.html;
}
```

> [!tip] Le dernier argument de try_files fait toute la différence
> `=404` renvoie une erreur, `/index.html` fait une redirection interne — voir [[Nginx 11 — root, alias & recettes de routing]] et [[Nginx 06 — Routing avec location & try_files]] pour le mécanisme général de `try_files`.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `fastcgi_pass` sans vérification d'existence du fichier | Toujours `try_files $uri =404;` avant |
| `proxy_pass` utilisé à la place de `fastcgi_pass` (ou l'inverse) | Vérifier le protocole attendu par le backend (HTTP vs FastCGI) |
| Supposer un ordre "top to bottom" des `location` | Connaître l'ordre réel : exact > préfixe le plus long (`^~` prioritaire) > regex dans l'ordre > repli préfixe |
| `ssl_protocols` incluant TLSv1/1.1 | Se limiter à `TLSv1.2 TLSv1.3` (profil Intermediate) |
| Slash final oublié/ajouté dans `proxy_pass` | Tester avec `curl -v`, voir Nginx 10 |
| `root`/`alias` confondus | `root` ajoute le chemin, `alias` le remplace |
| Backend redéployé, Nginx route vers l'ancienne IP | `resolver` + `set $backend` pour forcer la résolution périodique |
| SPA qui reçoit des 404 sur ses routes internes | `try_files ... /index.html` au lieu de `=404` |
