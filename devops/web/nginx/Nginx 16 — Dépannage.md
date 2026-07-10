#devops #nginx #dépannage #avancé

## Diagnostic en 30 secondes

```bash
# 1. Nginx tourne-t-il ?
systemctl status nginx

# 2. La configuration est-elle valide ?
sudo nginx -t

# 3. Dump complet de la configuration (tous les includes résolus)
sudo nginx -T | less

# 4. Logs systemd récents
journalctl -u nginx --since "10 min ago"

# 5. Logs d'erreur Nginx
sudo tail -20 /var/log/nginx/error.log

# 6. Le port est-il bien écouté ?
ss -tlnp | grep :80
```

`nginx -T` diffère de `nginx -t` : au-delà de la validation, il affiche la configuration **complète**, tous les `include` résolus — utile pour vérifier ce qui est réellement chargé quand plusieurs fichiers se combinent (voir [[Nginx 04 — Structure du fichier de configuration]]).

## Erreurs courantes et première piste

| Erreur | Cause probable | Première vérification |
|--------|--------------------|----------------------------|
| 403 Forbidden | Permissions de fichiers | `sudo chown -R www-data:www-data /var/www/` |
| 404 Not Found | Mauvais `root` ou fichier absent | Le chemin dans la config correspond-il au fichier réel ? |
| 502 Bad Gateway | Le backend ne répond pas | Voir la méthode ci-dessous |
| 504 Gateway Timeout | Backend trop lent | Augmenter `proxy_read_timeout` (voir [[Nginx 10 — Reverse proxy HTTP & WebSockets]]) |
| "Address already in use" | Port déjà occupé par un autre processus | `ss -tlnp \| grep :80` pour identifier le processus |

## Déboguer une 502 pas à pas

```bash
# 1. Le backend répond-il, indépendamment de Nginx ?
curl -I http://127.0.0.1:3000
# "Connection refused" → le backend est down, pas un problème Nginx

# 2. Nginx (l'utilisateur qui l'exécute) peut-il joindre le backend ?
sudo -u www-data curl http://127.0.0.1:3000
# Utile pour isoler un problème de permissions ou SELinux propre à cet utilisateur

# 3. Le detail exact de l'échec
sudo tail -f /var/log/nginx/error.log
# "connect() failed (111: Connection refused)" confirme le diagnostic de l'étape 1
```

> [!info] SELinux (RHEL/Rocky) bloque parfois le reverse proxy
> Si le backend répond bien en direct (`curl` réussit) mais que Nginx renvoie quand même 502, `sudo setsebool -P httpd_can_network_connect 1` autorise Nginx à initier des connexions réseau sortantes sous SELinux — une cause fréquente et non liée à la configuration Nginx elle-même.

## Recharger sans interruption

```bash
sudo nginx -t                # toujours tester d'abord
sudo systemctl reload nginx  # ou : sudo nginx -s reload
```

> [!warning] `reload` ≠ `restart`
> `reload` fait relire la configuration aux workers sans jamais couper les connexions en cours — `restart` arrête puis relance le service, avec une coupure de service même brève. Ne jamais utiliser `restart` en production quand `reload` suffit.

## Cas particuliers

> [!tip] La méthode reste la même quelle que soit l'erreur
> Service actif ? → Configuration valide ? → Que disent les logs ? — dans cet ordre, cette séquence identifie la cause de la grande majorité des incidents avant d'aller chercher plus loin. Voir aussi [[Serveurs Web — Checklist production & dépannage]] pour la version générique à tout serveur web.

> [!info] Pour les erreurs de certificat spécifiquement
> Chaîne de certificat incomplète, mauvais SNI, expiration — ces symptômes se diagnostiquent avec `openssl s_client` plutôt qu'avec les commandes ci-dessus, orientées configuration Nginx. Voir [[Nginx 12 — Certificats Let's Encrypt avec Certbot]].
