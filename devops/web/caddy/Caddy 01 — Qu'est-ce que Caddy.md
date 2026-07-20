#devops #caddy #fondamentaux

## Qu'est-ce que Caddy ?

**Caddy** est un serveur web moderne dont la caractéristique déterminante est le **HTTPS automatique par défaut** : contrairement à Nginx ou Apache, il obtient et renouvelle ses certificats Let's Encrypt sans configuration TLS explicite (voir [[Serveurs Web — Choisir son serveur]] pour la comparaison des trois).

## Historique : simplifier ce qui était complexe

Créé par **Matt Holt** en 2015, Caddy répond à un problème précis de son époque : obtenir un certificat SSL était complexe et coûteux. Écrit en **Go**, il se distribue comme un binaire unique — un déploiement plus simple qu'un serveur nécessitant des modules dynamiques compilés séparément.

## Caddy vs Nginx : quand choisir Caddy

| Critère | Caddy | Nginx |
|---------|-------|-------|
| HTTPS | Automatique (Let's Encrypt intégré) | Manuel (Certbot séparé — voir [[Nginx 12 — Certificats Let's Encrypt avec Certbot]]) |
| Configuration | Lisible (Caddyfile) | Plus verbeuse (`nginx.conf`) |
| Rechargement | `caddy reload` sans perte | `nginx -s reload` |
| HTTP/3 | Natif | Module externe |
| Courbe d'apprentissage | ~30 minutes | Plusieurs heures |

> [!tip] En résumé
> Caddy pour la simplicité et le HTTPS sans effort ; Nginx pour le contrôle fin et un écosystème plus mature (voir [[Serveurs Web — Choisir son serveur]]).

## Dans quel contexte utiliser Caddy

- Exposer une API ou un dashboard derrière HTTPS sans toucher à Certbot ni à un cron de renouvellement.
- Servir un site statique avec un Caddyfile de quelques lignes.
- Proxyfier plusieurs applications sur un même serveur, routées par domaine ou par chemin.
- Prototyper rapidement un environnement de développement avec des certificats auto-signés (`tls internal`, voir [[Caddy 05 — HTTPS automatique]]).
- Remplacer Nginx sur un petit serveur ou un homelab sans sacrifier les performances.

## Cas particuliers

> [!info] "Secure by default" est une philosophie, pas juste une fonctionnalité
> Le HTTPS automatique n'est pas une option qu'on active — c'est le comportement par défaut dès qu'un domaine public est déclaré. Cette inversion (sécurisé sauf action contraire, plutôt qu'ouvert sauf configuration) distingue Caddy des deux autres serveurs de ce vault dès la conception.
