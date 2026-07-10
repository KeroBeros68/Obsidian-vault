#devops #web #production #dépannage

## Pourquoi une checklist

Oublier un seul point avant une mise en production peut rendre un site inaccessible, piratable ou lent. Cette liste couvre les erreurs les plus fréquentes, quel que soit le serveur (Nginx, Apache, Caddy — voir [[Serveurs Web — Choisir son serveur]]).

## Les 7 points à vérifier avant la mise en production

**1. Ports et firewall**
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

**2. Permissions des fichiers**
```bash
# Le serveur web (www-data, nginx...) doit pouvoir lire les fichiers servis
sudo chown -R www-data:www-data /var/www/monsite
sudo chmod -R 755 /var/www/monsite
```

**3. TLS/HTTPS**
- Certificat valide (Let's Encrypt ou équivalent)
- Renouvellement automatique testé (`certbot renew --dry-run`)
- HSTS activé une fois le certificat validé en production

**4. Logs et rotation**
```bash
ls /etc/logrotate.d/nginx   # ou apache2 — vérifier que la rotation est bien configurée
```

**5. Reload sans coupure de service**
```bash
# Toujours valider la configuration avant de recharger
nginx -t && systemctl reload nginx
```

**6. Monitoring**
- Endpoint de statut exposé (`/nginx_status`, `/server-status`...)
- Export vers Prometheus si l'infrastructure le prévoit

**7. Hardening**
- Masquer la version du serveur (`server_tokens off;` — voir [[Nginx 09 — Sécurité de base]])
- En-têtes de sécurité (`X-Frame-Options`, `Content-Security-Policy`)
- Limiter les méthodes HTTP autorisées à celles réellement utilisées

## Dépannage express

Dans la majorité des cas, l'une de ces cinq erreurs explique le problème :

| Erreur | Ce que ça signifie | Première chose à vérifier |
|--------|------------------------|---------------------------------|
| 403 Forbidden | Le serveur refuse l'accès | `ls -la /var/www/...` — le serveur web peut-il lire ce fichier ? |
| 404 Not Found | Le fichier demandé n'existe pas | Le `root` de la configuration pointe-t-il vers le bon dossier ? |
| 502 Bad Gateway | Le backend ne répond pas | `curl http://127.0.0.1:PORT` — l'application tourne-t-elle réellement ? |
| 504 Gateway Timeout | Le backend répond trop lentement | Application surchargée, ou base de données lente |
| "Address already in use" | Un autre processus occupe déjà le port | `ss -tlnp \| grep :80` — qui occupe ce port ? |

## La méthode universelle de diagnostic

```bash
# 1. Le service tourne-t-il ?
systemctl status nginx   # ou apache2 / caddy

# 2. La configuration est-elle valide ?
nginx -t                 # ou apachectl configtest / caddy validate

# 3. Que disent les logs ?
tail -20 /var/log/nginx/error.log
```

Ces trois commandes, dans cet ordre, résolvent la grande majorité des incidents avant même d'aller chercher plus loin — voir [[Serveurs Web — Concepts fondamentaux]] pour le rôle des logs.

## Cas particuliers

> [!warning] Un reload sans test préalable peut casser la production
> `systemctl restart nginx` sans avoir validé la configuration au préalable (`nginx -t`) risque d'appliquer une configuration invalide et de laisser le service arrêté — toujours tester avant de recharger ou redémarrer.

> [!tip] 502 et 504 se distinguent par leur cause
> Un 502 signale que le backend ne répond pas du tout (arrêté, crashé, mauvais port). Un 504 signale qu'il répond, mais trop lentement pour le délai imparti — deux diagnostics différents malgré des symptômes proches côté utilisateur.
