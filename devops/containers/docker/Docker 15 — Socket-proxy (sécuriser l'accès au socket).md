#devops #docker #sécurité #avancé

## La preuve que `:ro` ne protège rien

[[Docker 08 — Sécurité des conteneurs]] établit qu'un accès au socket Docker équivaut à un accès root sur l'hôte. Le réflexe de le monter en `:ro` ne change rien : `:ro` porte sur le **fichier** socket (interdiction de le remplacer), pas sur le contenu des requêtes qui transitent dessus. Un `POST` d'arrêt de conteneur emprunte le même canal qu'un `GET` de liste — le système de fichiers ne fait aucune différence entre les deux.

Démonstration : arrêter un conteneur à travers un socket monté en lecture seule.

```bash
docker run -d --name cible nginx:alpine
CID=$(docker inspect -f '{{.Id}}' cible)

docker run --rm --user 0 -v /var/run/docker.sock:/var/run/docker.sock:ro \
  curlimages/curl -s -o /dev/null -w "%{http_code}\n" \
  -X POST --unix-socket /var/run/docker.sock \
  "http://localhost/v1.44/containers/$CID/stop"
# 204

docker ps -a --format '{{.Names}} {{.Status}}' | grep cible
# cible   Exited (0)
```

`HTTP 204`, conteneur arrêté — à travers un socket `:ro`. Le read-only y était une question de **confiance dans le code** de l'outil, pas une frontière technique : un bug ou une dépendance compromise transforme ce pouvoir latent en incident réel. Le moindre privilège exige de **retirer la capacité**, pas de compter sur la retenue de l'outil.

## Déployer un docker-socket-proxy

`tecnativa/docker-socket-proxy` se place devant le vrai socket, expose un endpoint TCP, et **refuse tout `POST` par défaut** — seuls les groupes d'endpoints explicitement déclarés sont ouverts.

```yaml
services:
  socketproxy:
    image: tecnativa/docker-socket-proxy:latest
    environment:
      - CONTAINERS=1
      - INFO=1
      - VERSION=1
      - EVENTS=1
      - IMAGES=1
      - NETWORKS=1
      - PING=1
      # POST non défini => 0 => toute écriture renvoie 403
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks: [proxynet]
    restart: unless-stopped

networks:
  proxynet: {}
```

```bash
docker compose up -d socketproxy
docker compose exec socketproxy wget -qO- http://localhost:2375/version | head -c 80
# {"Platform":{"Name":"Docker Engine..."}   -> lecture OK
```

## Brancher un outil via DOCKER_HOST

L'outil ne monte **plus** le socket réel — il pointe sur le proxy via une variable d'environnement standard, honorée par la plupart des SDK Docker (Go, Python) :

```yaml
monitoring:
  image: votre/outil:latest
  environment:
    - DOCKER_HOST=tcp://socketproxy:2375   # jamais le vrai socket
  networks: [proxynet]
  depends_on: [socketproxy]
```

L'outil découvre les conteneurs à travers le proxy, sans jamais accéder au socket réel.

## Vérifier l'isolation : le test qui compte

```bash
# Lecture : autorisée
docker compose exec socketproxy wget -qO- -S http://localhost:2375/version 2>&1 | grep HTTP
#   HTTP/1.1 200 OK

# Écriture : refusée
CID=$(docker inspect -f '{{.Id}}' un-conteneur)
docker compose exec socketproxy \
  wget -qO- -S --post-data='' "http://localhost:2375/containers/$CID/stop" 2>&1 | grep HTTP
#   HTTP/1.1 403 Forbidden
```

Le read-only n'est plus une promesse de l'outil : c'est une règle **imposée** par le proxy. Même compromis, l'outil ne peut pas agir sur les conteneurs.

## N'ouvrir que le strict nécessaire

| Variable | Ouvre | Pour quel usage |
|----------|-------|-------------------|
| `CONTAINERS` | Liste et inspection des conteneurs | Monitoring, découverte |
| `IMAGES` | Liste des images | Détection de mises à jour |
| `NETWORKS` | Réseaux | Découverte (ex. Traefik) |
| `EVENTS` | Flux d'événements | Réaction en temps réel |
| `INFO`, `VERSION`, `PING` | Métadonnées et santé du démon | Base, quasiment toujours nécessaire |

> [!warning] Les variables qui rouvrent l'écriture
> `POST=1` réautorise **toutes** les écritures — annule l'intérêt du proxy. Les drapeaux granulaires (`ALLOW_START`, `ALLOW_STOP`, `ALLOW_RESTARTS`, `EXEC`, `BUILD`, `COMMIT`) rouvrent des actions précises : à n'activer que si l'outil en question a réellement besoin d'agir (ex. un outil de déploiement), jamais par défaut pour un simple outil de lecture.

## Sécuriser le proxy lui-même

Le proxy monte le vrai socket : c'est lui, désormais, le composant de confiance root-equivalent — à traiter comme tel.

- **Réseau dédié, sans port publié** : le proxy et ses clients sur un réseau interne (`proxynet`) ; le port `2375` ne doit jamais être exposé sur l'hôte ou publiquement.
- **Durcissement du conteneur proxy** : `read_only: true`, `security_opt: [no-new-privileges:true]`, `cap_drop: [ALL]` (voir [[Docker 08 — Sécurité des conteneurs]] pour ces mécanismes en détail).
- **Un proxy par besoin** plutôt qu'un proxy unique tout-ouvert partagé par tous les outils — limite le rayon d'impact si un client est compromis.

## Cas particuliers

> [!info] Alternative plus durcie : wollomatic/socket-proxy
> Va plus loin que `tecnativa/docker-socket-proxy` : filtrage par regex des chemins, TLS mutuel, allowlist stricte. À envisager pour un usage particulièrement sensible.

> [!tip] Complémentaire au mode rootless, pas un substitut
> Le mode rootless (voir [[Docker 11 — Sous le capot (namespaces, cgroups, seccomp)]]) déplace le démon hors de root et réduit l'enjeu à la source. Un socket-proxy reste utile même en rootless, dès qu'un outil tiers a besoin d'un accès filtré à l'API plutôt qu'au socket brut.

> [!warning] Dépannage courant
> Outil qui ne voit rien : groupe d'endpoints manquant (souvent `CONTAINERS=1` oublié) ou `DOCKER_HOST` mal pointé. `403` sur une action attendue : l'outil tente une écriture — décider d'ouvrir un `ALLOW_*` précis, ou de retirer cette action de l'outil. `DOCKER_HOST` ignoré : certains outils codent en dur un chemin de socket ; vérifier s'ils acceptent `tcp://` ou une variable équivalente.
