#devops #nginx #tls #sécurité

## Terminer le TLS sur Nginx

```nginx
listen 443 ssl;
listen [::]:443 ssl;

ssl_certificate     /etc/nginx/ssl/__DOMAIN_NAME__.crt;
ssl_certificate_key /etc/nginx/ssl/__DOMAIN_NAME__.key;
ssl_protocols TLSv1.2 TLSv1.3;
```

Le paramètre `ssl` sur `listen` indique que ce port attend une connexion chiffrée. Le certificat et sa clé privée sont ensuite utilisés pour établir la session TLS avant que Nginx ne traite la requête HTTP en clair.

## Recommandations actuelles (profil "Intermediate", Mozilla SSL Config Generator — 2026)

| Élément | Recommandation |
|---------|-----------------|
| Protocoles | `TLSv1.2` et `TLSv1.3` uniquement — TLSv1.0/1.1 sont obsolètes et à proscrire |
| Ciphers TLS 1.2 | Restreindre aux suites AEAD (`ECDHE-...-GCM...`, `...-CHACHA20-POLY1305`) via `ssl_ciphers` |
| Ciphers TLS 1.3 | Non configurables côté Nginx — gérés directement par OpenSSL, déjà sécurisés par défaut |
| `ssl_dhparam` | Devenu optionnel pour un site standard ; à conserver seulement si des suites DHE doivent rester actives pour une contrainte de compatibilité héritée |

> [!info] Configuration à jour
> Ces réglages évoluent avec les recommandations de sécurité. Le [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) reste la référence à consulter pour une configuration générée et à jour au moment du déploiement.

## Cas particuliers

> [!warning] Certificat auto-signé vs certificat de confiance
> Un `ssl_certificate` auto-signé (comme un certificat généré localement) chiffre correctement la connexion, mais le navigateur du client affichera un avertissement car aucune autorité de certification reconnue ne l'a signé. En production, un certificat émis par une autorité reconnue (ex. Let's Encrypt) est nécessaire pour éviter cet avertissement.

> [!tip] Le port TLS n'est pas forcément 443
> Rien n'impose techniquement que `listen ... ssl` porte sur 443 — c'est une convention. Dans un environnement conteneurisé où Nginx est lui-même derrière un autre reverse proxy ou un mapping de ports Docker, le port interne peut différer du port exposé publiquement (illustré par `__NGINX_PORT__` dans une configuration templatisée).
