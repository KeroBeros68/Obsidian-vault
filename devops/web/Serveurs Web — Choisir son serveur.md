#devops #web #fondamentaux

## Qu'est-ce qu'un serveur web ?

Un serveur web reçoit les requêtes des clients (les requêtes HTTP), va chercher ce qu'il faut fournir (fichiers ou application), et renvoie une réponse. Sans lui, un site existe sur un disque mais reste inaccessible depuis Internet.

## Pourquoi plusieurs serveurs web différents ?

Chacun des trois grands serveurs web a été créé pour résoudre un problème précis à son époque :

| Année | Événement |
|-------|-----------|
| 1995 | Lancement d'Apache HTTP Server — dominant pendant 15 ans |
| 2004 | Nginx résout le problème C10K (10 000 connexions simultanées) — voir [[Nginx 01 — Qu'est-ce que Nginx]] |
| 2015 | Caddy introduit le HTTPS automatique par défaut |

Aujourd'hui, Nginx reste le plus utilisé devant les sites à fort trafic, Apache demeure très présent en hébergement mutualisé et PHP, et Caddy gagne du terrain pour sa simplicité.

## Choisir en fonction du besoin

| Besoin principal | Serveur recommandé | Pourquoi |
|----------------------|------------------------|----------|
| Reverse proxy + performance | **Nginx** | Architecture événementielle, faible RAM, cache intégré |
| HTTPS automatique | **Caddy** | Certificats Let's Encrypt sans configuration, HTTP/3 natif |
| Compatibilité legacy / `.htaccess` | **Apache** | Écosystème historique, modules dynamiques, PHP `mod_php` |
| Simplicité avant tout | **Caddy** | Configuration en quelques lignes, zéro complexité TLS |

## Tableau comparatif détaillé

| Critère | Nginx | Apache | Caddy |
|---------|-------|--------|-------|
| Architecture | Événementielle async | Processus/threads (MPM) | Événementielle (Go) |
| HTTPS | Manuel (Certbot) | Manuel (Certbot) | Automatique |
| Configuration | Fichiers `.conf` | Fichiers `.conf` + `.htaccess` | `Caddyfile` simple |
| HTTP/2 | Oui | Oui | Oui |
| HTTP/3 (QUIC) | Expérimental | Non | Natif |
| Reverse proxy | Excellent | Bon | Très bon |
| Cache | Intégré | Par modules | Basique |
| RAM | Très faible | Modérée à élevée | Faible |
| Cas idéal | Production haute charge | Legacy PHP, CMS | Petits/moyens déploiements |

> [!tip] Règle simple pour trancher rapidement
> Nouveau projet sans contrainte particulière → Caddy (HTTPS automatique, simplicité). Trafic important ou besoin de reverse proxy → Nginx (référence de l'industrie, déjà couvert par ce vault — voir [[Nginx — Index des fiches]]). WordPress, PHP legacy, dépendance à `.htaccess` → Apache.

## Cas particuliers

> [!info] Les trois génèrent le même résultat HTTP
> Quel que soit le serveur choisi, le protocole servi au client (HTTP/1.1, HTTP/2, HTTP/3) et les concepts sous-jacents restent identiques — voir [[Serveurs Web — Concepts fondamentaux]]. Le choix porte sur l'architecture interne, la simplicité de configuration et l'écosystème, pas sur des fonctionnalités HTTP différentes.

> [!warning] `.htaccess` n'existe que chez Apache
> Un fichier `.htaccess` copié depuis un tutoriel Apache n'a aucun effet sur Nginx ou Caddy — chacun a son propre format de configuration (`nginx.conf`, `Caddyfile`) et sa propre syntaxe pour les réécritures d'URL, les redirections, etc.

## Pour aller plus loin

Ce vault couvre les trois serveurs en détail : [[Nginx — Index des fiches]], [[Apache — Index des fiches]], [[Caddy — Index des fiches]]. Vocabulaire commun dans [[Serveurs Web — Glossaire]].
