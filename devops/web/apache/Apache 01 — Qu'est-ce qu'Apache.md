#devops #apache #fondamentaux

## Qu'est-ce qu'Apache HTTP Server ?

**Apache HTTP Server** est un serveur web open source qui sert des fichiers statiques (HTML, CSS, JS, images), héberge plusieurs sites sur une même machine (Virtual Hosts), gère HTTPS, et peut agir comme reverse proxy vers des applications backend — les mêmes rôles que Nginx (voir [[Serveurs Web — Choisir son serveur]]), avec une architecture et une logique de configuration différentes.

## Historique : une origine disputée, même par l'ASF

Créé en 1995 à partir des patchs appliqués au serveur NCSA HTTPd, Apache est devenu rapidement le serveur web le plus utilisé au monde ; Nginx l'a dépassé en parts de marché en 2019, mais Apache reste très présent dans les environnements PHP traditionnels et l'hébergement mutualisé.

> [!info] « A Patchy Server » : une étymologie plus incertaine qu'il n'y paraît
> L'histoire d'un nom né du jeu de mots « a patchy server » (un serveur fait de patchs) est largement répandue — la FAQ officielle du projet l'a elle-même relayée entre 1996 et 2001. Mais l'Apache Software Foundation présente aujourd'hui une origine différente : un hommage aux nations Apache, reconnues pour leurs stratégies de guerre et leur endurance. Les témoignages des créateurs eux-mêmes se contredisent d'une interview à l'autre. À traiter comme une anecdote disputée plutôt qu'un fait établi.

## Apache vs Nginx : quand choisir Apache

| Situation | Pourquoi Apache |
|-----------|---------------------|
| PHP avec `mod_php` | Combinaison simple, pas de FastCGI à configurer (voir [[Apache 02 — Architecture & MPM]] pour la contrainte associée) |
| `.htaccess` indispensable | Des CMS comme WordPress s'appuient dessus par défaut (voir [[Apache 06 — .htaccess & AllowOverride]]) |
| Écosystème déjà en place | Équipe déjà formée, modules ou configurations existantes à maintenir |
| `mod_rewrite` avancé | Réécritures d'URL très riches, documentation abondante |

Hors de ces cas, Nginx ou Caddy restent souvent plus simples et plus performants — voir [[Serveurs Web — Choisir son serveur]] pour le comparatif complet des trois.

## Les 3 concepts à maîtriser avant tout le reste

| Concept | Rôle |
|---------|------|
| **Virtual Host** | Un fichier de configuration qui associe un domaine (`ServerName`) à un dossier de fichiers (`DocumentRoot`) — voir [[Apache 05 — Virtual Hosts]] |
| **Modules** | Apache ne charge que le strict nécessaire par défaut ; chaque fonctionnalité (TLS, réécriture, proxy) s'active explicitement — voir [[Apache 03 — Modules (a2enmod)]] |
| **MPM** (*Multi-Processing Module*) | La façon dont Apache traite les requêtes en parallèle (un processus par requête, ou des threads partagés) — voir [[Apache 02 — Architecture & MPM]] |

## Cas particuliers

> [!tip] Retenir Apache par ce qu'il faut activer, pas par ce qu'il fait par défaut
> Contrairement à un serveur qui embarquerait tout, la première question face à une configuration Apache inconnue est « quels modules sont chargés ? » (`apache2ctl -M` / `httpd -M`) plutôt que de chercher une fonctionnalité qui semble absente alors qu'elle est juste désactivée.
