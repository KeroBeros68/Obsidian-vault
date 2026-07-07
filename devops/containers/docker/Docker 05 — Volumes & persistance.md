#devops #docker #volumes #persistance

## Pourquoi persister des données

Un conteneur est éphémère : sa couche en lecture-écriture disparaît avec lui (voir [[Docker 01 — Images vs Conteneurs]]). Pour des données qui doivent survivre — bases de données, fichiers uploadés, configuration — Docker propose trois types de montage.

## Les trois types de montage

| Type | Géré par | Cas d'usage |
|------|----------|-------------|
| **Volume** (nommé) | Docker (`/var/lib/docker/volumes/`) | Données de production : bases de données, fichiers persistants |
| **Bind mount** | L'utilisateur (chemin hôte explicite) | Développement : code source édité en temps réel |
| **tmpfs** | RAM uniquement (Linux) | Données sensibles ou temporaires : tokens, cache jetable |

Ces trois types restent **locaux** à l'hôte qui exécute le conteneur. Pour du stockage réseau partagé entre plusieurs hôtes (NFS, CIFS, plugins), voir [[Docker 13 — Volumes réseau & plugins (NFS, CIFS, SSHFS)]].

```bash
# Volume nommé — Docker gère l'emplacement
docker run -d -v db_data:/var/lib/mysql mysql:latest

# Bind mount — chemin hôte explicite
docker run -d -v /home/user/site:/usr/share/nginx/html nginx:latest

# tmpfs — en mémoire, jamais sur disque
docker run -d --tmpfs /app/cache redis:latest
```

## Syntaxe --mount en détail

`--mount` est plus verbeux que `-v`/`--tmpfs`, mais explicite chaque paramètre et échoue avec une erreur claire si la source d'un bind mount n'existe pas — préférable en production et dans les Dockerfiles.

```bash
docker run --mount type=volume,source=mon-volume,target=/app/data nginx
docker run --mount type=bind,source=/chemin/hote,target=/app/data nginx
docker run --mount type=volume,source=mon-volume,target=/app/data,readonly nginx
docker run --mount type=volume,source=mon-volume,target=/app/data,volume-nocopy nginx
```

| Option `--mount` | Effet |
|--------------------|-------|
| `readonly` | Monte en lecture seule — équivalent du suffixe `:ro` en syntaxe `-v` |
| `volume-nocopy` | Empêche Docker de copier le contenu existant du conteneur vers le volume à sa création (comportement par défaut sans cette option) |

## Volumes nommés vs anonymes

```bash
docker run -v /app/cache nginx              # ❌ anonyme : nom en hash aléatoire, difficile à cibler
docker run -v cache_nginx:/app/cache nginx  # ✅ nommé : identifiable, gérable explicitement
```

Un volume **anonyme** (chemin de destination seul, sans nom avant le `:`) reçoit un identifiant en hash généré par Docker. Il survit à `docker rm` seul, mais disparaît avec `docker rm -v` ou `docker volume prune` — sans jamais apparaître clairement dans `docker volume ls` sous un nom exploitable.

> [!warning] `docker system prune` ne touche pas aux volumes, `docker volume prune` supprime aussi les anonymes
> `docker system prune` (voir [[Docker 10 — Configuration production & nettoyage]]) laisse les volumes intacts, nommés ou anonymes. `docker volume prune`, lui, supprime **tous** les volumes non référencés — y compris des volumes anonymes qui contenaient encore des données utiles mais dont plus aucun conteneur ne pointe vers eux.

## Comportement et subtilités

- **Volume** : entièrement isolé du système hôte, portable entre environnements, peut être partagé entre plusieurs conteneurs.
- **Bind mount** : tout changement de fichier côté hôte est immédiatement visible dans le conteneur, et inversement — pratique pour le hot-reload en développement, mais dépend de la structure de l'hôte (chemin qui peut ne pas exister ailleurs).
- **tmpfs** : si le conteneur s'arrête, le contenu disparaît immédiatement ; ne fonctionne que sur hôte Linux.

## Durcir un montage tmpfs

```bash
# Limite de taille + interdiction d'exécuter des binaires, d'élever les privilèges, ou de créer des devices
docker run --tmpfs /tmp:noexec,nosuid,nodev,size=100m nginx

# Équivalent --mount, avec un mode de permissions explicite
docker run --mount type=tmpfs,destination=/tmp,tmpfs-size=100m,tmpfs-mode=1777 nginx
```

| Option | Effet | Pourquoi |
|--------|-------|----------|
| `noexec` | Interdit l'exécution de binaires depuis ce montage | Empêche d'exécuter un fichier malveillant déposé dans `/tmp` |
| `nosuid` | Ignore les bits `setuid`/`setgid` | Empêche une élévation de privilèges via un binaire monté |
| `nodev` | Interdit les fichiers de périphériques | Réduit la surface d'attaque |
| `size=100m` | Plafonne l'usage RAM de ce montage | Évite qu'un tmpfs non borné ne sature la mémoire de l'hôte |
| `mode=1777` | Permissions avec sticky bit | Utile pour un `/tmp` partagé entre plusieurs utilisateurs dans le conteneur |

En Docker Compose :

```yaml
services:
  app:
    image: nginx:alpine
    tmpfs:
      - /tmp:size=100M,mode=1777
      - /var/cache:size=200M,noexec,nosuid
```

```yaml
# Pattern courant en développement
volumes:
  - ./src:/app/src           # bind mount : code source édité à chaud
  - node-modules:/app/node_modules  # volume : dépendances (rapide, surtout sur Mac/Windows)
```

## Volume nommé "épinglé" à un chemin hôte

```yaml
volumes:
  mariadb_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /home/user/data/mariadb
```

| Clé | Rôle |
|-----|------|
| `driver: local` | Le driver de volume par défaut de Docker — explicite ici car indispensable pour pouvoir fournir des `driver_opts` |
| `driver_opts.type` | Type de filesystem à monter, au sens de la commande `mount` ; `none` signifie qu'aucun filesystem réseau ou spécifique n'est utilisé, seulement une opération de montage local |
| `driver_opts.o` | Options de montage, au sens de `mount -o` ; `bind` déclenche un bind mount classique vers le chemin indiqué par `device` |
| `driver_opts.device` | Chemin réel sur l'hôte vers lequel pointe le volume |

Ce pattern combine les deux mondes : Compose gère `mariadb_data` comme un volume nommé classique (référencé par son nom dans les services, visible via `docker volume ls`), mais son contenu réel est **épinglé à un chemin précis de l'hôte** plutôt que placé dans l'emplacement interne de Docker (`/var/lib/docker/volumes/`). En interne, ces trois options reproduisent exactement ce que ferait `mount --bind <device> /var/lib/docker/volumes/mariadb_data/_data` — Docker se contente d'exécuter ce montage à la création du volume. Utile pour garder la portabilité de nom d'un volume nommé tout en sachant exactement où se trouvent les données sur l'hôte (ex. pour un script de sauvegarde externe).

Quand plusieurs services ont chacun besoin de ce traitement, la même structure se répète, seul `device:` change :

```yaml
volumes:
  mariadb_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /home/user/data/mariadb
  wordpress_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /home/user/data/wordpress
```

> [!info] `type: none` + `o: bind`, pas un vrai filesystem réseau
> `type: none` indique qu'aucun filesystem réseau n'est monté ; `o: bind` déclenche un bind mount classique vers `device:`. Le résultat se comporte comme un bind mount côté données, mais reste géré comme un volume côté commandes Docker (`docker volume ls`, `docker volume rm`...).

> [!warning] Le chemin `device` doit exister et être accessible
> Contrairement à un bind mount direct dans `services.volumes` (qui peut créer silencieusement un dossier manquant avec `-v`), une erreur dans `device:` ou des permissions insuffisantes sur ce chemin fait échouer la création du volume, pas seulement son utilisation.

## Sauvegarder et restaurer un volume

```bash
# Sauvegarder : monte le volume et le dossier courant dans un conteneur jetable
docker run --rm -v mon-volume:/data -v $(pwd):/backup alpine \
  tar czf /backup/data.tar.gz /data

# Restaurer : extrait l'archive dans le volume cible
docker run --rm -v mon-volume:/data -v $(pwd):/backup alpine \
  tar xzf /backup/data.tar.gz -C /
```

Ce pattern ne dépend d'aucun outil externe : un conteneur `alpine` jetable monte à la fois le volume à sauvegarder et un dossier hôte, exécute `tar`, puis se supprime (`--rm`) une fois terminé.

> [!info] Outils dédiés pour une stratégie de sauvegarde récurrente
> Le pattern `tar` manuel convient à une sauvegarde ponctuelle. Pour une stratégie automatisée et récurrente, des outils dédiés existent : **Restic** (sauvegarde chiffrée et incrémentale), **Rclone** (synchronisation vers un stockage cloud), **Offen/docker-volume-backup** (spécifiquement pensé pour les volumes Docker, avec rotation automatique).

## Organiser ses volumes avec des labels

```bash
docker volume create --label env=production --label app=wordpress db_data
docker volume ls --filter label=env=production   # cibler par label plutôt que par nom
```

Un label n'a aucun effet fonctionnel sur le volume — il sert uniquement à le filtrer ou le documenter, utile dès qu'un hôte accumule des volumes de plusieurs projets ou environnements.

## Nettoyer les volumes inutilisés

```bash
docker volume ls                 # lister tous les volumes
docker volume rm mon-volume      # supprimer un volume précis (doit être inutilisé)
docker volume prune              # supprimer tous les volumes non montés dans un conteneur actif
```

> [!warning] `docker volume prune` est irréversible
> Cette commande supprime tous les volumes **non référencés par au moins un conteneur** (actif ou arrêté), sans distinction entre "temporaire" et "oublié mais important" — vérifier `docker volume ls` et la sortie de confirmation avant de valider.

## Cas particuliers

> [!warning] Jamais de bind mount sensible en production
> Monter `/`, `/etc` ou `/var` de l'hôte dans un conteneur expose le système hôte à des risques de sécurité sérieux si le conteneur est compromis.

> [!tip] Règle simple
> Volume par défaut pour toute donnée persistante · Bind mount seulement pour le code source en dev · tmpfs pour ce qui ne doit jamais toucher le disque.

> [!info] -v vs --mount
> `-v` est plus court, `--mount` est plus explicite et recommandé en production car il donne des messages d'erreur plus clairs et ne crée pas silencieusement un dossier manquant.
