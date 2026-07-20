#devops #caddy #fondamentaux

## Site statique minimal

```bash
sudo mkdir -p /var/www/monsite
echo '<h1>Hello Caddy!</h1>' | sudo tee /var/www/monsite/index.html
sudo chown -R caddy:caddy /var/www/monsite
```

```caddyfile
# Pour un test local, sans TLS
:8080 {
    root * /var/www/monsite
    file_server
}
```

```bash
sudo systemctl reload caddy
curl http://localhost:8080
```

## Avec un domaine public : HTTPS automatique

```caddyfile
monsite.com {
    root * /var/www/monsite
    file_server
}
```

Rien de plus : si le DNS de `monsite.com` pointe vers ce serveur, Caddy obtient automatiquement un certificat Let's Encrypt, redirige HTTP → HTTPS, et renouvelle le certificat avant expiration — voir [[Caddy 05 — HTTPS automatique]] pour le détail du mécanisme et ses prérequis.

## SPA (Single Page Application)

```caddyfile
monsite.com {
    root * /var/www/spa
    try_files {path} /index.html
    file_server
}
```

`try_files {path} /index.html` teste d'abord le chemin demandé (`{path}`), puis retombe sur `index.html` si rien ne correspond — laissant le routeur côté client (React Router, Vue Router...) gérer la route, comme le pattern équivalent chez Nginx (voir [[Nginx 11 — root, alias & recettes de routing]]).

## Cas particuliers

> [!warning] Permissions du dossier servi
> Le processus Caddy s'exécute en tant qu'utilisateur `caddy` — un `chown -R caddy:caddy` sur le dossier servi est nécessaire, sinon `file_server` renvoie une erreur de permission plutôt que le contenu attendu.

> [!info] `file_server browse` pour un listing de répertoire
> Ajouter `browse` après `file_server` affiche la liste des fichiers d'un dossier sans `index.html` — utile pour un partage de fichiers ponctuel, à éviter sur un site de production qui ne le souhaite pas explicitement.
