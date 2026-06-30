#devops #docker #volumes #persistance

## Pourquoi persister des données

Un conteneur est éphémère : sa couche en lecture-écriture disparaît avec lui (voir [[Docker 01 — Images vs Conteneurs]]). Pour des données qui doivent survivre — bases de données, fichiers uploadés, configuration — Docker propose trois types de montage.

## Les trois types de montage

| Type | Géré par | Cas d'usage |
|------|----------|-------------|
| **Volume** (nommé) | Docker (`/var/lib/docker/volumes/`) | Données de production : bases de données, fichiers persistants |
| **Bind mount** | L'utilisateur (chemin hôte explicite) | Développement : code source édité en temps réel |
| **tmpfs** | RAM uniquement (Linux) | Données sensibles ou temporaires : tokens, cache jetable |

```bash
# Volume nommé — Docker gère l'emplacement
docker run -d -v db_data:/var/lib/mysql mysql:latest

# Bind mount — chemin hôte explicite
docker run -d -v /home/user/site:/usr/share/nginx/html nginx:latest

# tmpfs — en mémoire, jamais sur disque
docker run -d --tmpfs /app/cache redis:latest
```

## Comportement et subtilités

- **Volume** : entièrement isolé du système hôte, portable entre environnements, peut être partagé entre plusieurs conteneurs.
- **Bind mount** : tout changement de fichier côté hôte est immédiatement visible dans le conteneur, et inversement — pratique pour le hot-reload en développement, mais dépend de la structure de l'hôte (chemin qui peut ne pas exister ailleurs).
- **tmpfs** : si le conteneur s'arrête, le contenu disparaît immédiatement ; ne fonctionne que sur hôte Linux.

```yaml
# Pattern courant en développement
volumes:
  - ./src:/app/src           # bind mount : code source édité à chaud
  - node-modules:/app/node_modules  # volume : dépendances (rapide, surtout sur Mac/Windows)
```

## Cas particuliers

> [!warning] Jamais de bind mount sensible en production
> Monter `/`, `/etc` ou `/var` de l'hôte dans un conteneur expose le système hôte à des risques de sécurité sérieux si le conteneur est compromis.

> [!tip] Règle simple
> Volume par défaut pour toute donnée persistante · Bind mount seulement pour le code source en dev · tmpfs pour ce qui ne doit jamais toucher le disque.

> [!info] -v vs --mount
> `-v` est plus court, `--mount` est plus explicite et recommandé en production car il donne des messages d'erreur plus clairs et ne crée pas silencieusement un dossier manquant.
