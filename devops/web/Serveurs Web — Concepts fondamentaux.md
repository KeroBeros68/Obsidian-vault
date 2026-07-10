#devops #web #fondamentaux

## Six concepts universels

Quel que soit le serveur choisi (Nginx, Apache, Caddy — voir [[Serveurs Web — Choisir son serveur]]), ces six concepts s'appliquent de la même façon. Une fois compris, ils se transposent d'un serveur à l'autre malgré des syntaxes de configuration différentes.

## 1. Requête/réponse HTTP

Le navigateur envoie une **requête** (« je veux la page `/contact` »), le serveur renvoie une **réponse** (« voici le contenu » ou « page introuvable »).

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
```

```
HTTP/1.1 200 OK
Content-Type: text/html

<!DOCTYPE html><html><body><h1>Hello!</h1></body></html>
```

| Code | Signification | Exemple concret |
|------|-----------------|--------------------|
| 200 | OK | Page affichée normalement |
| 301/302 | Redirection | L'URL a changé, suivre ce lien |
| 403 | Interdit | Accès refusé (permissions ou config) |
| 404 | Introuvable | Le fichier n'existe pas |
| 500 | Erreur serveur | Bug applicatif ou de configuration |
| 502 | Mauvaise passerelle | Le backend derrière le reverse proxy ne répond pas |

## 2. Virtual hosts : plusieurs sites sur un serveur

Plusieurs domaines partagent la même adresse IP ; le serveur route selon l'en-tête `Host:` de la requête — équivalent applicatif de plusieurs enregistrements DNS pointant vers la même IP.

```nginx
server {
    server_name blog.exemple.fr;
    root /var/www/blog;
}
server {
    server_name shop.exemple.fr;
    root /var/www/shop;
}
```

Un visiteur demandant `blog.exemple.fr` reçoit le contenu de `/var/www/blog` ; `shop.exemple.fr` reçoit `/var/www/shop` — même IP, même port, contenu différent. Implémentation détaillée pour Nginx dans [[Nginx 05 — Server blocks & virtual hosting]].

## 3. Reverse proxy : le serveur web comme intermédiaire

Une application (Node.js, Python, Go...) tourne en interne sur un port applicatif (ex. 3000), mais les utilisateurs attendent le port 80/443. Le serveur web reçoit la requête publique et la transmet au backend.

```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
}
```

Sans reverse proxy : `http://monsite.com:3000` (port à retenir, peu présentable). Avec reverse proxy : `https://monsite.com` (propre, HTTPS géré par le serveur web). Ce vault détaille une variante FastCGI (PHP-FPM plutôt qu'un port HTTP direct) dans [[Nginx 07 — Reverse proxy vers un backend (FastCGI)]].

## 4. HTTPS/TLS : chiffrer les communications

Sans chiffrement, n'importe quel intermédiaire réseau peut intercepter les données échangées, y compris des mots de passe en clair.

| Solution | Difficulté | Renouvellement |
|----------|--------------|-------------------|
| Caddy | Automatique | Automatique |
| Certbot + Nginx/Apache | Quelques minutes de configuration | Automatique via `cron` |
| Certificat géré manuellement | Complexe | Manuel tous les 90 jours |

Détails TLS pour Nginx (protocoles recommandés, ciphers) dans [[Nginx 08 — TLS-SSL]].

## 5. Cache et compression

- **Cache** : évite de recalculer une réponse à chaque visite, en la servant directement depuis une copie déjà prête.
- **Compression** (gzip) : réduit la taille des données transmises sur le réseau.

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

| Sans optimisation | Avec optimisation | Gain |
|-----------------------|------------------------|------|
| Page de 500 Ko | 150 Ko (gzip) | -70% de bande passante |
| Recalculée à chaque visite | Servie depuis le cache | -90% de temps serveur |

## 6. Logs : comprendre ce qui se passe

| Fichier | Contenu | Quand le consulter |
|---------|---------|------------------------|
| `access.log` | Toutes les requêtes reçues | Analyser le trafic, détecter des anomalies |
| `error.log` | Les erreurs du serveur | Systématiquement dès qu'un problème survient |

```bash
tail -20 /var/log/nginx/error.log   # dernières erreurs
tail -f /var/log/nginx/error.log    # suivre en temps réel
```

> [!tip] Le réflexe à avoir avant tout autre diagnostic
> Face à un comportement inattendu, consulter `error.log` en premier résout la majorité des cas — avant d'aller chercher plus loin dans la configuration ou le code applicatif. Voir [[Serveurs Web — Checklist production & dépannage]] pour la méthode complète.

## Cas particuliers

> [!warning] Un reverse proxy transmet la requête, il ne la remplace pas
> `proxy_set_header Host $host` (ou l'équivalent FastCGI `fastcgi_param HTTP_HOST $http_host`, voir [[Nginx 07 — Reverse proxy vers un backend (FastCGI)]]) est nécessaire pour que le backend connaisse le domaine d'origine — sans ce transfert explicite, l'application peut générer des URLs incorrectes.
