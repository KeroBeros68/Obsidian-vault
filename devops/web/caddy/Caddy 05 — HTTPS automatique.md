#devops #caddy #tls #intermédiaire

## Comment ça marche

Quand un domaine public est déclaré (`example.com { }`), Caddy :

1. Détecte que c'est un domaine public (pas `localhost`, pas une IP).
2. Contacte Let's Encrypt pour prouver le contrôle du domaine (challenge HTTP).
3. Obtient un certificat valide 90 jours.
4. Configure TLS avec des options de sécurité modernes par défaut.
5. Crée une redirection HTTP → HTTPS automatique.
6. Renouvelle le certificat environ 30 jours avant expiration.

Tout cela sans écrire la moindre directive TLS — contrairement à Nginx/Certbot (voir [[Nginx 12 — Certificats Let's Encrypt avec Certbot]]) ou Apache/Certbot (voir [[Apache 10 — Certificats Let's Encrypt avec Certbot]]), où l'obtention du certificat reste une étape manuelle séparée de la configuration du serveur.

## Prérequis (le HTTPS auto n'est pas magique)

| Condition | Pourquoi |
|-----------|----------|
| Le domaine pointe vers le serveur (DNS A/AAAA) | Let's Encrypt vérifie la possession du domaine |
| Ports 80 et 443 accessibles depuis Internet | Nécessaires au challenge HTTP |
| Le domaine est public | `localhost` ou une IP brute ne déclenchent pas l'obtention automatique |

## Les 3 modes TLS

| Mode | Quand l'utiliser | Configuration |
|------|----------------------|-------------------|
| **Auto** (défaut) | Domaine public accessible | `example.com { ... }` — rien à ajouter |
| **Internal** | Réseau local, développement | `tls internal` |
| **Manuel** | Certificats déjà émis par ailleurs | `tls /chemin/cert.pem /chemin/key.pem` |

## tls internal pour un réseau local

```caddyfile
intranet.local {
    tls internal
    reverse_proxy localhost:3000
}
```

Caddy génère un certificat auto-signé via sa propre autorité de certification interne.

```bash
caddy trust   # installe la CA Caddy dans le trust store système (Linux/macOS)
```

> [!warning] Sans `caddy trust`, les navigateurs affichent un avertissement
> Un certificat auto-signé par la CA interne de Caddy n'est pas reconnu par défaut par les navigateurs/clients — installer cette CA dans le trust store de chaque machine cliente supprime l'avertissement, sans quoi le certificat reste valide mais signalé comme non fiable.

## DNS Challenge : pour un serveur sans port 80/443 exposé

Si le firewall ou le NAT empêche l'accès direct aux ports 80/443 (le challenge HTTP classique est alors impossible), le DNS Challenge valide la possession du domaine autrement — indispensable aussi pour un certificat wildcard (`*.example.com`).

```bash
# Compiler Caddy avec un plugin DNS (ex. Cloudflare)
xcaddy build --with github.com/caddy-dns/cloudflare
```

```caddyfile
*.example.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:3000
}
```

## Cas particuliers

> [!warning] Domaine `.local`/`.lan` sans `tls internal`
> Sans cette directive, Caddy tente d'obtenir un certificat Let's Encrypt pour un domaine qui n'est pas résolvable publiquement — la tentative échoue systématiquement (voir [[Caddy 09 — Dépannage]] pour le diagnostic ACME).

> [!info] Rate limit Let's Encrypt
> Trop de tentatives d'obtention de certificat pour un même domaine dans une courte période déclenche une limitation temporaire côté Let's Encrypt — un délai d'attente (généralement autour d'une heure) est alors nécessaire avant de retenter, plutôt que de changer immédiatement la configuration.
