#devops #caddy #pièges #erreurs #debugging

## 🪤 Piège 1 — DNS pas encore propagé, HTTPS auto échoue

```
❌ Déclarer example.com { } avant que le DNS ne pointe réellement vers le serveur
```

```bash
# ✅ Vérifier avant de déployer
dig example.com
```

> [!warning] Caddy ne peut pas obtenir de certificat pour un domaine mal résolu
> Voir [[Caddy 05 — HTTPS automatique]] et [[Caddy 09 — Dépannage]] pour le diagnostic ACME complet.

---

## 🪤 Piège 2 — Ports 80/443 fermés

```
❌ Firewall ou NAT bloquant les ports 80/443 : Let's Encrypt ne peut pas valider le domaine
```

> [!tip] Mémo
> Ouvrir les ports, ou utiliser le DNS Challenge si l'exposition directe n'est pas possible — voir [[Caddy 05 — HTTPS automatique]].

---

## 🪤 Piège 3 — Domaine local sans tls internal

```caddyfile
# ❌ Caddy tente d'obtenir un certificat Let's Encrypt pour un domaine non public
intranet.local {
    reverse_proxy localhost:3000
}
```

```caddyfile
# ✅
intranet.local {
    tls internal
    reverse_proxy localhost:3000
}
```

> [!warning] `.local`/`.lan` nécessitent explicitement tls internal
> Voir [[Caddy 05 — HTTPS automatique]].

---

## 🪤 Piège 4 — Oublier handle_path pour retirer un préfixe

```caddyfile
# ❌ Le backend reçoit /api/users, alors qu'il attend /users
reverse_proxy /api/* localhost:3000
```

```caddyfile
# ✅
handle_path /api/* {
    reverse_proxy localhost:3000
}
```

> [!tip] Mémo
> Voir [[Caddy 04 — Reverse proxy]] — même piège conceptuel que le slash final dans `proxy_pass` chez Nginx.

---

## 🪤 Piège 5 — Ordre des handlers incorrect

```caddyfile
# ❌ file_server capture tout avant que /api/* ne soit évalué
example.com {
    file_server
    reverse_proxy /api/* localhost:3000
}
```

```caddyfile
# ✅ chemins spécifiques avant les handlers génériques
example.com {
    reverse_proxy /api/* localhost:3000
    file_server
}
```

> [!warning] Un handler large placé trop tôt masque tout le reste
> Voir [[Caddy 04 — Reverse proxy]].

---

## 🪤 Piège 6 — `https://` dans reverse_proxy sans nécessité

```caddyfile
# ❌ Échoue souvent si le backend ne parle qu'en HTTP local
reverse_proxy https://localhost:3000
```

```caddyfile
# ✅
reverse_proxy localhost:3000
```

> [!tip] Mémo
> N'ajouter `https://` que si le backend exige réellement TLS en interne.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| DNS non propagé | `dig example.com` avant de déployer |
| Ports 80/443 fermés | Ouvrir les ports ou utiliser le DNS Challenge |
| Domaine `.local`/`.lan` sans `tls internal` | Ajouter `tls internal` |
| Préfixe non retiré avant le backend | `handle_path /api/* { ... }` |
| Handler générique placé avant un chemin spécifique | Chemins précis d'abord, `file_server` catch-all en dernier |
| `https://` inutile dans `reverse_proxy` | Ne l'ajouter que si le backend l'exige réellement |
