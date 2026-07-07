#devops #docker #compose #orchestration

## Docker Compose

Compose décrit une application multi-conteneurs dans un seul fichier YAML. Chaque `service` correspond à un conteneur, avec son image (ou son Dockerfile à builder), ses volumes, ses réseaux et ses dépendances.

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - db
    networks:
      - frontend

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  pgdata:
```

## Commandes essentielles

| Situation | Commande |
|-----------|----------|
| Démarrer tous les services | `docker compose up -d` |
| Arrêter les services (garde les volumes) | `docker compose down` |
| Arrêter et supprimer les volumes | `docker compose down -v` |
| Voir les logs d'un service | `docker compose logs -f web` |
| Exécuter une commande dans un service | `docker compose exec web bash` |

## Isolation entre services

```yaml
services:
  web:
    networks: [frontend]   # web ne peut PAS atteindre db directement
  api:
    networks: [frontend, backend]
  db:
    networks: [backend]    # db n'est accessible que depuis 'backend'
```

Compose crée un réseau bridge dédié par projet (voir [[Docker 06 — Réseaux]]) : tous les services qui partagent un réseau peuvent se joindre par leur **nom de service**, sans configuration DNS manuelle.

## Politique de redémarrage (restart)

| Valeur | Comportement |
|--------|--------------|
| `no` (défaut) | Ne redémarre jamais automatiquement, même après un crash |
| `always` | Redémarre systématiquement — y compris après un redémarrage du démon Docker ou de l'hôte, **même si le conteneur avait été arrêté manuellement** |
| `on-failure[:N]` | Redémarre uniquement si le conteneur se termine avec un code de sortie non nul (échec) ; `N` optionnel limite le nombre de tentatives |
| `unless-stopped` | Redémarre systématiquement, **sauf** si le conteneur a été explicitement arrêté (`docker stop` / `docker compose stop`) — cet arrêt manuel reste respecté même après un redémarrage du démon |

```yaml
services:
  api:
    build: .
    restart: unless-stopped   # ✅ choix par défaut recommandé en production

  worker:
    build: .
    restart: on-failure:5     # retente 5 fois max en cas de crash, puis abandonne

  db:
    image: postgres:17
    restart: always           # redémarre même après un arrêt manuel suivi d'un reboot du démon
```

> [!warning] always vs unless-stopped : la nuance sur l'arrêt manuel
> Avec `always`, arrêter un conteneur manuellement (`docker stop`) puis redémarrer le démon Docker (reboot de la machine) relance quand même le conteneur — l'arrêt manuel n'est pas mémorisé au-delà d'un redémarrage du démon. Avec `unless-stopped`, cet arrêt manuel est mémorisé : le conteneur ne repart pas tout seul tant qu'il n'a pas été explicitement relancé (`docker start` / `docker compose up`).

> [!tip] unless-stopped, le choix le plus sûr par défaut en production
> Il combine la résilience (redémarrage après crash ou reboot de l'hôte) avec un arrêt manuel qui reste effectif — évite la surprise d'un conteneur qui "revient tout seul" juste après une opération de maintenance.

> [!info] restart ne s'applique pas à un service Swarm
> Sous Docker Swarm, la politique de redémarrage passe par `deploy.restart_policy` (avec ses propres clés `condition`, `delay`, `max_attempts`), pas par `restart:` — ce dernier n'a d'effet qu'en Compose "classique" ou avec `docker run`.

> [!warning] restart ne réagit pas à un état "unhealthy"
> Une politique de redémarrage ne se déclenche que lorsque le **conteneur s'arrête** (crash, exit code non nul). Un conteneur marqué `unhealthy` par son `HEALTHCHECK` (voir [[Docker 03 — Dockerfile]]) continue de tourner tel quel — `restart: always` ne le relance pas tout seul. Pour redémarrer automatiquement un conteneur unhealthy, il faut Docker Swarm ou un outil dédié (ex. `willfarrell/autoheal`, qui surveille l'état de santé et déclenche lui-même le `docker restart`).

## Healthcheck et depends_on par condition

Un `healthcheck` définit une commande exécutée périodiquement dans le conteneur pour vérifier qu'il est réellement opérationnel — pas seulement démarré.

```yaml
services:
  db:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 15s   # délai de grâce avant de compter un premier échec

  api:
    build: .
    depends_on:
      db:
        condition: service_healthy   # attend que le healthcheck de db passe au vert
```

| Forme de `depends_on` | Attend quoi |
|------------------------|-------------|
| `depends_on: [db]` (liste simple) | Que le **conteneur** `db` démarre — rien sur son état interne |
| `depends_on: db: condition: service_started` | Équivalent explicite de la forme liste |
| `depends_on: db: condition: service_healthy` | Que le `healthcheck` de `db` réponde "healthy" |

> [!warning] "Démarré" ne veut pas dire "prêt"
> `depends_on: [db]` déclenche `api` dès que le conteneur `db` est lancé — pas quand la base accepte réellement des connexions. Un service qui se connecte à sa dépendance dès son propre démarrage peut planter au premier essai. `condition: service_healthy` est le seul mécanisme Compose natif qui attend une réelle disponibilité, et nécessite un `healthcheck` défini sur le service dont on dépend.

## Secrets Compose

```yaml
services:
  db:
    image: mariadb:11
    secrets:
      - db_root_password
    environment:
      - MARIADB_ROOT_PASSWORD_FILE=/run/secrets/db_root_password

secrets:
  db_root_password:
    file: ./secrets/db_root_password.txt
```

Chaque secret déclaré est monté en lecture seule sous `/run/secrets/<nom>`, uniquement dans les services qui le référencent explicitement. C'est plus sûr que de passer un mot de passe en clair via `environment:`, visible avec `docker inspect` ou `docker top`, et potentiellement figé dans une couche d'image si mal utilisé au build.

> [!info] Convention `_FILE`
> Beaucoup d'images officielles (MariaDB, Postgres, WordPress...) acceptent une variante `_FILE` de leurs variables d'environnement (`MARIADB_ROOT_PASSWORD_FILE`) qui pointe vers un fichier à lire, plutôt que la valeur en clair — pensée pour fonctionner directement avec les secrets Docker.

## Logging et ancres YAML

Par défaut, le driver de logs `json-file` n'a aucune limite de taille : les logs d'un conteneur bavard peuvent remplir le disque de l'hôte au fil du temps.

```yaml
x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  web:
    image: nginx
    logging: *default-logging   # réutilise le bloc défini plus haut

  api:
    build: .
    logging: *default-logging
```

`x-logging` est une **ancre YAML** : un bloc nommé (préfixé `x-` par convention Compose pour signaler "extension, pas une clé Compose standard") réutilisable ailleurs avec `*default-logging`, évitant de dupliquer la même configuration dans chaque service.

> [!warning] json-file sans limite = disque qui se remplit
> Sans `max-size`/`max-file`, un conteneur peut écrire des logs indéfiniment. Toujours fixer une limite en production, via ce bloc `logging` ou une configuration par défaut au niveau du démon (`/etc/docker/daemon.json`).

## Cas particuliers

> [!warning] version: est obsolète
> Le champ `version: "3.8"` en haut du fichier n'a plus d'effet depuis Compose v2 — il est ignoré et génère un avertissement. Ne plus l'inclure dans les nouveaux fichiers.

> [!warning] down -v supprime les données
> `docker compose down -v` supprime aussi les volumes nommés déclarés dans le fichier — donc les données persistées avec eux. À utiliser uniquement en connaissance de cause.

> [!tip] docker compose, pas docker-compose
> La commande moderne s'écrit avec un espace (`docker compose`), pas un tiret. L'ancien binaire autonome `docker-compose` est obsolète et n'est plus maintenu.

> [!tip] build.args sans valeur = lu depuis le shell ou .env
> `args: - MARIADB_PORT` (sans `=valeur`) transmet au build la valeur de la variable d'environnement du shell ou du fichier `.env` portant le même nom — équivalent à `--build-arg MARIADB_PORT=$MARIADB_PORT` en ligne de commande. Voir [[Docker 03 — Dockerfile]] pour le côté `ARG` dans le Dockerfile.
