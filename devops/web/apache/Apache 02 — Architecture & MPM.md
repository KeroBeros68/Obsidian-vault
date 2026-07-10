#devops #apache #architecture #fondamentaux

## MPM : comment Apache traite les requêtes en parallèle

**MPM** (*Multi-Processing Module*) désigne le mécanisme par lequel Apache gère plusieurs requêtes simultanées — contrairement à Nginx, dont le modèle événementiel unique est fixe (voir [[Nginx 02 — Fonctionnement interne]]), Apache choisit parmi plusieurs MPM interchangeables.

| MPM | Fonctionnement | Comportement |
|-----|-------------------|-------------------|
| **prefork** | Un processus dédié par requête | Simple et très compatible, mais gourmand en RAM |
| **worker** | Processus + threads, plusieurs threads par processus | Plus économe que prefork |
| **event** | Comme worker, avec une gestion dédiée des connexions keepalive en attente | Le plus efficace des trois pour beaucoup de connexions |

## Quel MPM choisir

| Situation | MPM recommandé | Pourquoi |
|-----------|---------------------|----------|
| PHP via `mod_php` | **prefork** | `mod_php` n'est pas thread-safe (voir plus bas) |
| PHP-FPM ou `proxy_fcgi` | **event** | Meilleure gestion des connexions, PHP tourne dans un processus séparé |
| Reverse proxy | **event** | Meilleure gestion des connexions keepalive |
| Beaucoup de connexions simultanées | **event** | Plus efficace en mémoire |
| Compatibilité maximale, contrainte inconnue | **prefork** | Fonctionne avec pratiquement tous les modules |

> [!warning] `mod_php` + MPM `event` = plantages aléatoires
> `mod_php` charge l'interpréteur PHP directement dans les processus Apache, mais son code n'est pas conçu pour être exécuté par plusieurs threads en parallèle (non thread-safe). Utiliser `mod_php` avec `worker` ou `event` peut provoquer des plantages imprévisibles. Vérifier `apache2ctl -M | grep php` : la présence de `php_module` signale `mod_php`, qui impose de rester sur `prefork`. Passer par PHP-FPM (`proxy_fcgi`) lève cette contrainte et permet d'utiliser `event`.

## Activer le MPM event

```bash
# Debian/Ubuntu
sudo a2dismod mpm_prefork
sudo a2enmod mpm_event
sudo systemctl restart apache2
```

```bash
# RHEL/Rocky — éditer /etc/httpd/conf.modules.d/00-mpm.conf
# Commenter LoadModule mpm_prefork_module
# Décommenter LoadModule mpm_event_module
sudo systemctl restart httpd
```

## Cas particuliers

> [!info] Le MPM se change globalement, pas par Virtual Host
> Contrairement à un module métier (proxy, rewrite...), le MPM est une propriété du processus Apache tout entier — impossible d'avoir `prefork` pour un site et `event` pour un autre sur la même instance. Un changement de MPM impacte tous les sites hébergés.

> [!tip] Vérifier le MPM actif
> `apache2ctl -V | grep MPM` (ou `httpd -V`) indique le MPM compilé/chargé actuellement — utile avant de diagnostiquer un problème de performance ou de compatibilité avec `mod_php`.
