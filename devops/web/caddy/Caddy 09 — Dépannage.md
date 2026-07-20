#devops #caddy #dépannage #avancé

## Commandes essentielles

```bash
caddy validate --config /etc/caddy/Caddyfile   # valider avant de recharger
sudo systemctl reload caddy                     # recharger sans downtime
sudo journalctl -u caddy -f                     # logs en temps réel
caddy fmt --overwrite /etc/caddy/Caddyfile      # reformater le fichier
```

Même discipline que `nginx -t`/`apache2ctl configtest` avant un reload (voir [[Nginx 16 — Dépannage]], [[Apache 13 — Dépannage]]) : toujours valider avant d'appliquer.

## Erreurs courantes

| Erreur | Cause probable | Solution |
|--------|--------------------|----------|
| `ACME challenge failed` | DNS incorrect ou ports 80/443 fermés | Vérifier le DNS, ouvrir les ports |
| 502 Bad Gateway | Le backend ne répond pas | `curl localhost:3000` pour tester |
| `too many redirects` | Boucle HTTP ↔ HTTPS | Vérifier la configuration proxy/CDN en amont |
| `permission denied` | Caddy ne peut pas lire les fichiers | `chown -R caddy:caddy /var/www` |
| `address already in use` | Un autre service occupe déjà le port | `ss -tlnp \| grep :80` |

## Déboguer une erreur ACME

```bash
sudo journalctl -u caddy | grep -i acme
```

| Message | Signification |
|---------|-------------------|
| "no valid IP addresses" | Le DNS n'est pas configuré ou pas encore propagé |
| "connection refused" | Le port 80 est fermé côté firewall |
| "rate limit" | Trop de tentatives d'obtention de certificat — attendre environ 1h avant de retenter |

## Déboguer un 502

```bash
# 1. Le backend tourne-t-il, indépendamment de Caddy ?
curl -I http://localhost:3000

# 2. Caddy (l'utilisateur qui l'exécute) peut-il joindre le backend ?
sudo -u caddy curl http://localhost:3000

# 3. Regarder les logs en direct
sudo journalctl -u caddy -f
```

Même démarche que pour Nginx (voir [[Nginx 16 — Dépannage]]) : isoler d'abord si le problème vient du backend ou de Caddy lui-même.

## Cas particuliers

> [!warning] Ordre des handlers : un handler large capture tout avant la règle spécifique
> Un `file_server` catch-all placé avant un `reverse_proxy /api/*` intercepte les requêtes destinées à l'API — toujours placer les chemins spécifiques avant les handlers génériques, voir [[Caddy 04 — Reverse proxy]].

> [!info] Domaine local sans tls internal
> Une tentative d'obtention de certificat Let's Encrypt pour un domaine `.local`/`.lan` échoue systématiquement — ajouter `tls internal` (voir [[Caddy 05 — HTTPS automatique]]) plutôt que de chercher une cause réseau.
