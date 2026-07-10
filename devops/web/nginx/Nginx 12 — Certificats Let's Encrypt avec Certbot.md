#devops #nginx #tls #avancé

## De la configuration TLS au certificat réel

[[Nginx 08 — TLS-SSL]] couvre la configuration TLS elle-même (`ssl_protocols`, `ssl_certificate`...). Cette fiche couvre l'obtention et le renouvellement pratiques d'un certificat gratuit via **Let's Encrypt** et **Certbot**, l'outil qui automatise ce processus.

## Installer Certbot

```bash
# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx

# RHEL/Rocky
sudo dnf install epel-release
sudo dnf install certbot python3-certbot-nginx
```

## Obtenir un certificat

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot modifie automatiquement la configuration Nginx existante pour y ajouter le certificat, le bloc HTTPS et une redirection HTTP → HTTPS :

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;   # forcé explicitement, ne pas se fier aux défauts (voir Nginx 08)
    ssl_prefer_server_ciphers on;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    location / {
        root /var/www/html;
        index index.html;
    }
}

server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

## Renouvellement automatique

Let's Encrypt émet des certificats valides 90 jours — Certbot installe un timer systemd qui gère le renouvellement sans intervention.

```bash
sudo certbot renew --dry-run     # simule un renouvellement sans le faire réellement
systemctl list-timers | grep certbot   # confirme que le timer est actif
```

## HSTS : quand l'activer

`Strict-Transport-Security` force les navigateurs à n'utiliser que HTTPS pour ce domaine, y compris pour les visites suivantes.

> [!warning] N'activer HSTS qu'après avoir confirmé que HTTPS fonctionne pleinement
> Un HSTS activé prématurément peut bloquer l'accès au site si HTTPS a un problème de configuration — les navigateurs refuseront alors de retomber en HTTP, y compris pour du débogage. Commencer avec un `max-age` court (`3600` = 1h) le temps de valider, puis passer à `31536000` (1 an) une fois la configuration confirmée stable.

## Cas particuliers

> [!info] `ssl_protocols` explicite malgré Certbot
> Certbot ne force pas systématiquement `TLSv1.2 TLSv1.3` selon la version — voir [[Nginx 08 — TLS-SSL]] pour la recommandation actuelle (profil Mozilla Intermediate) et [[Nginx — Pièges classiques]] pour le risque de laisser des protocoles obsolètes actifs par défaut.

> [!tip] Certificat en échec : d'abord vérifier le DNS et le port 80
> Certbot valide la possession du domaine via une requête HTTP sur le port 80 (challenge HTTP-01) — un certificat qui échoue à l'obtention est très souvent dû à un DNS pas encore propagé ou un port 80 fermé par le firewall, avant même d'être un problème de configuration Nginx.
