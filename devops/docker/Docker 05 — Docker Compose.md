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

Compose crée un réseau bridge dédié par projet (voir [[Docker 04 — Réseaux]]) : tous les services qui partagent un réseau peuvent se joindre par leur **nom de service**, sans configuration DNS manuelle.

## Cas particuliers

> [!warning] version: est obsolète
> Le champ `version: "3.8"` en haut du fichier n'a plus d'effet depuis Compose v2 — il est ignoré et génère un avertissement. Ne plus l'inclure dans les nouveaux fichiers.

> [!warning] down -v supprime les données
> `docker compose down -v` supprime aussi les volumes nommés déclarés dans le fichier — donc les données persistées avec eux. À utiliser uniquement en connaissance de cause.

> [!tip] docker compose, pas docker-compose
> La commande moderne s'écrit avec un espace (`docker compose`), pas un tiret. L'ancien binaire autonome `docker-compose` est obsolète et n'est plus maintenu.
