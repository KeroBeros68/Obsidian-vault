#devops #apache #tls #avancé

## Installer Certbot

```bash
# Debian/Ubuntu
sudo apt install certbot python3-certbot-apache
```

```bash
# RHEL/Rocky
sudo dnf install epel-release
sudo dnf install certbot python3-certbot-apache
```

## Obtenir le certificat

```bash
sudo certbot --apache -d exemple.com -d www.exemple.com
```

Certbot vérifie la possession du domaine, obtient le certificat, **modifie automatiquement le Virtual Host** pour y ajouter les directives TLS, et configure la redirection HTTP → HTTPS — équivalent du comportement de `certbot --nginx` (voir [[Nginx 12 — Certificats Let's Encrypt avec Certbot]]), avec le plugin `python3-certbot-apache` à la place.

## Vérifier le renouvellement automatique

```bash
sudo certbot renew --dry-run
```

Un timer systemd gère le renouvellement automatique, environ 30 jours avant l'expiration du certificat (valide 90 jours chez Let's Encrypt).

## Après Certbot : durcir manuellement la configuration TLS

Certbot pose un certificat fonctionnel, mais pas nécessairement la configuration TLS la plus stricte — ajouter explicitement les directives modernes (voir [[Apache 08 — TLS-SSL]]) :

```apache
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
SSLHonorCipherOrder off
```

## Quand activer HSTS

```apache
Header always set Strict-Transport-Security "max-age=31536000"
```

> [!warning] N'activer HSTS qu'une fois HTTPS confirmé fonctionnel
> Un navigateur qui a reçu ce header refuse ensuite tout accès en HTTP à ce domaine, y compris pour du débogage — activer seulement après avoir vérifié que le certificat et la redirection fonctionnent pleinement. Commencer avec un `max-age` court le temps de valider, avant de passer à `31536000` (1 an) en production — même précaution que pour Nginx (voir [[Nginx 12 — Certificats Let's Encrypt avec Certbot]]).

## Cas particuliers

> [!warning] Activer `mod_ssl` avant Certbot, pas après
> Une cause fréquente d'échec silencieux : lancer `certbot --apache` avant `a2enmod ssl` (voir [[Apache 03 — Modules (a2enmod)]]) — Certbot a besoin que le module soit déjà chargé pour pouvoir modifier la configuration TLS du Virtual Host.

> [!tip] Générateur de configuration TLS
> Le [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) reste la référence pour une configuration `SSLProtocol`/`SSLCipherSuite` à jour, adaptée à la version exacte d'Apache et OpenSSL installée.
