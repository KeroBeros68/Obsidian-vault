#devops #docker #dockerfile

## Le Dockerfile

Un Dockerfile est un fichier texte contenant une suite d'instructions. Chaque instruction crée une nouvelle couche d'image, exécutée dans l'ordre du fichier.

```dockerfile
FROM python:3.12-slim      # image de base
WORKDIR /app                # dossier de travail dans l'image
COPY requirements.txt .     # copie un fichier précis
RUN pip install -r requirements.txt --no-cache-dir
COPY . .                    # copie le reste du code
CMD ["python", "app.py"]    # commande lancée au démarrage du conteneur
```

## Cache de build : l'ordre compte

Docker compare chaque instruction à son historique de build. Si une instruction et ses entrées sont identiques à une couche en cache, Docker réutilise le cache. Mais si une instruction change, Docker invalide le cache de cette instruction et de toutes celles qui suivent — elles devront être reconstruites.

| Situation | Syntaxe / Approche |
|-----------|-------------------|
| Dépendances qui changent rarement | Les `COPY` + `RUN install` en premier |
| Code source qui change souvent | Le `COPY . .` du code en dernier |
| Fichiers à exclure du contexte de build | `.dockerignore` |

```dockerfile
# ❌ Mauvais ordre : un changement de code invalide aussi l'install des deps
COPY . .
RUN pip install -r requirements.txt

# ✅ Bon ordre : les deps restent en cache si requirements.txt n'a pas changé
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

## Cache mounts BuildKit (--mount=type=cache)

Le cache de couches (vu ci-dessus) invalide tout le cache d'une instruction dès qu'un fichier d'entrée change. Un **cache mount** est différent : c'est un cache persistant **entre builds**, monté uniquement pendant l'exécution d'un `RUN`, qui ne fait jamais partie de la couche finale — utile pour éviter de retélécharger les mêmes dépendances à chaque build, même quand le fichier de dépendances change.

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./

RUN --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev
```

```dockerfile
# Python (pip)
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir -r requirements.txt

# Go modules
RUN --mount=type=cache,target=/go/pkg/mod \
    go build -o /app/main .

# Debian/Ubuntu (apt)
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt \
    apt-get update && apt-get install -y --no-install-recommends curl
```

> [!info] La directive # syntax= est obligatoire
> `# syntax=docker/dockerfile:1` en toute première ligne du Dockerfile active la syntaxe étendue de BuildKit — sans elle, `--mount=type=cache` (comme `--mount=type=secret`, voir [[Secrets 04 — .dockerignore & hygiène de build]]) est rejeté.

> [!tip] Vérifier que BuildKit est actif
> Sur les versions récentes de Docker, BuildKit est activé par défaut. Si le cache semble ignoré, `export DOCKER_BUILDKIT=1` avant `docker build` force son activation sur une installation plus ancienne ou reconfigurée.

## Multi-stage build

Une seule image de build (avec compilateurs, outils, dépendances de dev) sert à fabriquer l'artefact ; une image finale minimale ne récupère que ce dont l'exécution a besoin.

```dockerfile
# Stage 1 — build
FROM golang:1.22-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server ./cmd/server

# Stage 2 — runtime minimal
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

Le binaire final ne transporte ni le compilateur Go, ni le code source — seulement l'exécutable.

## Trois stages : un stage base partagé

Au-delà de deux stages, un stage intermédiaire `base` (dépendances runtime communes) évite de dupliquer son contenu entre le stage de build et le stage final, tout en gardant les outils de compilation hors de l'image livrée.

```dockerfile
# ❌ Un seul stage : apk del supprime les paquets, mais chaque RUN reste une couche distincte
FROM alpine:3.20
RUN apk update && apk upgrade
RUN apk add --no-cache python3 openssl
RUN apk add --no-cache --virtual .build-deps python3-dev gcc musl-dev openssl-dev
RUN pip3 install --no-cache-dir ansible
RUN apk del .build-deps
# → l'image finale fait ~393 Mo : les couches d'installation des outils de build
#   restent présentes dans l'historique de l'image, même supprimées ensuite
```

```dockerfile
# ✅ Trois stages : le stage builder (avec gcc, python3-dev...) n'existe jamais dans l'image finale
FROM alpine:3.20 AS base
RUN apk add --no-cache python3 openssl

FROM base AS builder
RUN apk add --no-cache --virtual .build-deps python3-dev gcc musl-dev openssl-dev
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip3 install --no-cache-dir ansible

FROM base
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
# → l'image finale fait ~165 Mo : seul /opt/venv est copié depuis builder,
#   le stage builder et ses outils de compilation ne sont jamais dans l'historique de l'image finale
```

> [!info] Même principe que le piège apt-get, poussé plus loin
> Le premier exemple retombe sur le même problème que dans « Nettoyer apt dans la même couche » (voir Cas particuliers) : `apk del .build-deps` dans un `RUN` séparé ne supprime rien du poids réel de l'image, la couche d'installation reste dans l'historique. Fusionner ces `RUN` en une seule instruction réglerait ce cas précis, mais le multi-stage règle le problème plus largement : le stage `builder` entier — pas seulement une commande — n'apparaît jamais dans l'image finale.

## ARG vs ENV

Deux instructions permettent de paramétrer un Dockerfile, mais leur portée est différente.

| | `ARG` | `ENV` |
|---|-------|-------|
| Disponible pendant le build | ✅ | ✅ |
| Disponible dans le conteneur à l'exécution | ❌ | ✅ |
| Valeur modifiable au build (`docker build --build-arg`) | ✅ | ❌ |
| Valeur modifiable au lancement (`docker run -e`) | ❌ | ✅ |

```dockerfile
# Disponible seulement pendant le build, absent du conteneur final
ARG DEBIAN_FRONTEND=noninteractive

# Une ARG peut alimenter une instruction qui a un effet à l'exécution, ici EXPOSE
ARG MARIADB_PORT=3306
EXPOSE ${MARIADB_PORT}
```

> [!info] ARG avant FROM
> Une `ARG` déclarée **avant** la première `FROM` n'est visible que dans cette ligne `FROM` (utile pour paramétrer le tag de l'image de base) ; elle disparaît ensuite sauf à être redéclarée après `FROM`.

## ENTRYPOINT vs CMD

| Forme | Comportement |
|-------|--------------|
| `CMD` seul | Commande par défaut, **entièrement remplacée** si `docker run image autre-chose` est utilisé |
| `ENTRYPOINT` seul | Commande fixe ; les arguments passés à `docker run` sont **ajoutés à sa suite** |
| `ENTRYPOINT` + `CMD` | `ENTRYPOINT` = exécutable fixe, `CMD` fournit ses arguments **par défaut**, remplaçables individuellement |

Le combo des deux est très courant pour une image qui doit exécuter une préparation (initialisation de données, attente d'une dépendance, génération de config) avant de lancer le vrai processus :

```dockerfile
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["mariadbd"]
```

```bash
#!/bin/bash
set -e

# Préparation : initialiser le datadir si le volume est vide, etc.
if [ ! -d "/var/lib/mysql/mysql" ]; then
    mariadb-install-db --user=mysql --datadir=/var/lib/mysql
fi

# Remplace le shell (PID 1) par la commande reçue en argument
exec "$@"
```

> [!warning] Sans `exec`, les signaux passent mal
> Si le script se termine par `"$@"` sans `exec`, la commande tourne comme processus **enfant** du script shell : le script reste PID 1 et les signaux envoyés par `docker stop` (SIGTERM) lui sont adressés à lui, pas à l'application, qui peut ne jamais recevoir l'ordre d'arrêt propre. `exec "$@"` remplace le processus du script par celui de la commande, qui devient PID 1 et reçoit directement les signaux.

## SHELL — changer l'interpréteur des formes shell

Les instructions en forme "shell" (`RUN commande`, sans crochets) s'exécutent par défaut via `/bin/sh -c` sous Linux. `SHELL` change cet interpréteur pour toute la suite du Dockerfile.

```dockerfile
SHELL ["/bin/bash", "-c"]
RUN [[ -f /etc/passwd ]] && echo "ok"   # syntaxe bash, invalide en /bin/sh POSIX strict
```

> [!tip] Utile pour activer pipefail
> `SHELL ["/bin/bash", "-c", "-o", "pipefail"]` fait échouer un `RUN` dès qu'une commande intermédiaire d'un pipe échoue (`cmd1 | cmd2`) — par défaut, seul le code de sortie de la dernière commande du pipe est pris en compte, ce qui peut masquer un échec silencieux.

## HEALTHCHECK — vérification de santé intégrée à l'image

Contrairement au `healthcheck:` de Compose (voir [[Docker 07 — Docker Compose]]), qui s'ajoute par-dessus une image existante, `HEALTHCHECK` peut être **intégré directement dans l'image** — il s'applique alors même à un simple `docker run`, sans fichier Compose.

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 --start-period=10s \
    CMD curl -f http://localhost/ || exit 1
```

| Option | Rôle |
|--------|------|
| `--interval` | Délai entre deux vérifications (défaut 30s) |
| `--timeout` | Délai avant de considérer une vérification comme un échec (défaut 30s) |
| `--retries` | Nombre d'échecs consécutifs avant de marquer le conteneur "unhealthy" (défaut 3) |
| `--start-period` | Délai de grâce au démarrage avant de compter les échecs (défaut 0s) |

```dockerfile
HEALTHCHECK NONE   # désactive un HEALTHCHECK hérité d'une image de base
```

> [!info] Compose peut surcharger celui de l'image
> Un `healthcheck:` défini dans un fichier Compose remplace entièrement celui inscrit dans l'image — utile pour ajuster les délais à un contexte de déploiement précis sans reconstruire l'image.

> [!warning] HEALTHCHECK n'est pas lu par Kubernetes
> `HEALTHCHECK` ne pilote l'état healthy/unhealthy et les redémarrages que sous Docker Engine, Compose ou Swarm. Sous Kubernetes, ce sont les probes du manifeste (`readinessProbe`, `livenessProbe`, `startupProbe`) qui font ce rôle — Kubernetes **ignore silencieusement** un `HEALTHCHECK` présent dans l'image. En contexte Kubernetes, considérer `HEALTHCHECK` comme un simple bonus pour le développement local (`docker run` isolé), pas comme la vérification réellement utilisée en production.

```bash
docker ps
# CONTAINER ID   STATUS
# abc123         Up 5 minutes (healthy)

docker inspect --format '{{json .State.Health}}' mon_conteneur | jq   # historique détaillé des checks
```

## STOPSIGNAL — signal envoyé à l'arrêt

Par défaut, `docker stop` envoie `SIGTERM`, puis `SIGKILL` si le conteneur ne s'arrête pas dans le délai imparti. `STOPSIGNAL` change le premier signal envoyé.

```dockerfile
STOPSIGNAL SIGQUIT
```

> [!info] Utile pour les applications qui attendent un signal différent
> Certains serveurs applicatifs (nginx avec `SIGQUIT` pour un arrêt "propre" des workers, par exemple) attendent un signal spécifique pour s'arrêter proprement. `STOPSIGNAL` évite un `docker stop` qui bascule directement sur un arrêt forcé (`SIGKILL`) faute de réponse au bon signal.

## VOLUME — point de montage déclaré dans l'image

`VOLUME` marque un chemin comme destiné à contenir des données externes au conteneur — à ne pas confondre avec la clé `volumes:` d'un fichier Compose (voir [[Docker 05 — Volumes & persistance]]), qui configure le montage réel.

```dockerfile
VOLUME ["/var/lib/mysql"]
```

Sans montage explicite (`-v` ou `volumes:`) au lancement du conteneur, Docker crée automatiquement un **volume anonyme** pour ce chemin — les données survivent donc à la suppression du conteneur, même si personne n'a rien déclaré au `docker run`.

> [!warning] Les écritures après VOLUME dans le même Dockerfile sont perdues à l'exécution
> Toute instruction placée **après** `VOLUME` dans le Dockerfile qui écrit à cet emplacement (`RUN echo ... > /var/lib/mysql/init`) voit ses changements **ignorés au lancement du conteneur** : le chemin est remplacé par le contenu du volume (anonyme ou nommé) dès le démarrage, pas par ce qui a été écrit pendant le build. Toute initialisation de données à cet emplacement doit passer par un script d'entrée exécuté au démarrage du conteneur (voir la section ENTRYPOINT vs CMD ci-dessus), pas par une instruction du Dockerfile.

> [!tip] Un volume anonyme reste jusqu'à nettoyage explicite
> `docker rm` seul ne supprime pas les volumes anonymes créés pour satisfaire une instruction `VOLUME` — `docker volume prune` ou `docker rm -v` sont nécessaires pour les éliminer, sans quoi ils s'accumulent silencieusement sur l'hôte.

## LABEL — métadonnées de l'image

`LABEL` attache des métadonnées clé/valeur à l'image, sans effet sur son contenu ni son exécution — consultables via `docker inspect`.

```dockerfile
LABEL maintainer="team@example.com" \
      version="1.0" \
      org.opencontainers.image.source="https://github.com/org/repo"
```

Les labels d'une image de base sont hérités par toute image construite dessus ; si une même clé est redéfinie, la valeur la plus récente l'emporte.

> [!tip] Convention OpenContainers
> Les clés préfixées `org.opencontainers.image.*` (`source`, `revision`, `licenses`...) suivent une convention standardisée, reconnue par de nombreux outils (registries, scanners) — à préférer à des clés maison pour les métadonnées courantes.

## ONBUILD — instructions différées à l'image dérivée

`ONBUILD` enregistre une instruction qui ne s'exécute **pas** au build de cette image, mais au build de toute image qui la prend comme base (`FROM cette-image`).

```dockerfile
FROM debian:stable
ONBUILD RUN apt-get update && apt-get install -y python3
```

Toute image dérivée de cette base exécutera automatiquement ce `RUN` lors de son propre build, sans que son Dockerfile ait besoin de le répéter.

> [!info] Cas d'usage réel : images de base partagées en équipe
> `ONBUILD` a du sens quand une équipe maintient une image de base commune et veut imposer une étape systématique (installation d'un agent de monitoring, copie d'un certificat interne) à toute image qui en hérite, sans documentation supplémentaire à suivre.

> [!warning] Effet caché pour qui lit l'image dérivée
> Un Dockerfile qui se contente de `FROM notre-image-de-base` déclenche des instructions invisibles à sa propre lecture — `docker history` ou la documentation de l'image de base restent nécessaires pour savoir ce qui s'exécute réellement.

## Choisir son image de base

Le choix de la distribution de base a un impact direct sur la taille de l'image, la surface d'attaque (nombre de CVE potentielles) et les problèmes de compatibilité rencontrés.

| Image de base | Taille approx. | Gestionnaire de paquets | Points d'attention |
|----------------|-----------------|--------------------------|---------------------|
| `scratch` | 0 Mo | aucun | Image totalement vide — uniquement pour un binaire compilé statiquement (Go, Rust) ; rien d'autre n'est présent, pas même les certificats CA |
| `gcr.io/distroless/*` | ~2-20 Mo | aucun | Pas de shell ni de package manager en usage normal ; le tag `:debug` ajoute un shell BusyBox pour un débogage ponctuel |
| `cgr.dev/chainguard/*` (Wolfi) | ~2-12 Mo | `apk` (Wolfi) | glibc + reconstruction quotidienne visant 0 CVE connu à la publication — voir cas particuliers |
| variantes `-alpine` (`node:20-alpine`) | ~5-50 Mo | `apk` | libc **musl**, pas glibc — voir avertissement ci-dessous |
| variantes `-slim` (`python:3.12-slim`) | ~50-80 Mo | `apt` | Debian allégé, mêmes bibliothèques ; certains paquets système "évidents" sont absents (`curl`, `git`...) |
| `debian:12`, `ubuntu:24.04` | ~120-180 Mo | `apt` | glibc standard, compatibilité maximale — le choix le plus sûr par défaut |

> [!warning] Alpine n'est pas juste "Debian en plus petit"
> Alpine remplace **glibc par musl libc**, pas seulement `apt` par `apk`. Un paquet qui embarque un binaire précompilé pour glibc (modules natifs Node.js, wheels Python taguées `manylinux`) peut échouer silencieusement ou nécessiter une recompilation depuis les sources sur Alpine — avec des temps de build parfois bien plus longs, voire des erreurs de segmentation difficiles à diagnostiquer.

> [!tip] Alpine convient bien aux binaires statiques
> Les langages qui produisent un binaire statique (Go, Rust avec la cible `musl`) n'ont aucune dépendance dynamique à la libc : Alpine (voire distroless ou `scratch`) devient alors un choix sans compromis, comme dans l'exemple multi-stage Go ci-dessus.

> [!warning] scratch : aucun certificat, aucune libc
> Une image `scratch` ne contient rien d'autre que ce qui y est copié explicitement — pas même les certificats CA nécessaires pour valider une connexion HTTPS sortante. Un binaire qui fait des appels HTTPS échouera silencieusement, sauf à copier `/etc/ssl/certs/ca-certificates.crt` depuis le stage de build, ou à utiliser `distroless/static` qui les inclut déjà.
>
> ```dockerfile
> FROM golang:1.22-alpine AS builder
> WORKDIR /app
> COPY . .
> RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/myapp
>
> FROM scratch
> COPY --from=builder /app/myapp /myapp
> COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
> ENTRYPOINT ["/myapp"]
> ```

> [!tip] distroless :debug pour déboguer ponctuellement
> `gcr.io/distroless/java21-debian12:debug` (ou l'équivalent pour un autre runtime) ajoute un shell BusyBox absent de l'image de production standard — pratique pour un `docker exec` de diagnostic ponctuel, à ne jamais utiliser comme image de production.

> [!info] Chainguard/Wolfi : 0 CVE ne veut pas dire "sans risque futur"
> Les images Chainguard (distribution Wolfi, construite avec l'outil `apko`) sont reconstruites quotidiennement pour viser 0 CVE connu **au moment de la publication** — une nouvelle vulnérabilité peut toujours être découverte après coup sur un paquet donné. L'avantage réel est la vitesse de reconstruction et de correction, pas une garantie figée dans le temps.

> [!info] Debian reste le choix par défaut raisonnable
> En cas de doute ou pour une image contenant un service système classique (base de données, serveur applicatif avec dépendances compilées), `debian:12` évite les mauvaises surprises liées à musl. Le gain de taille d'Alpine, Chainguard ou distroless ne vaut la complexité que si l'image est reconstruite/déployée très fréquemment ou si la taille/surface d'attaque est une contrainte forte (edge, CI, production exposée).

## Guide de choix par langage

| Langage | Image recommandée | Pourquoi |
|---------|---------------------|----------|
| Go / Rust | `scratch` ou `distroless/static` | Binaire compilé statiquement, aucune dépendance dynamique |
| Java | `distroless/java21` ou `chainguard/jre` | JRE minimal, pas de shell nécessaire à l'exécution |
| Python | `chainguard/python` ou `python:3.12-slim` | glibc pour les extensions natives (numpy, pandas...) — voir l'avertissement Alpine ci-dessus |
| Node.js | `chainguard/node` ou `node:20-slim` | Évite les problèmes de compatibilité musl des dépendances natives npm |
| .NET | `mcr.microsoft.com/dotnet/runtime` | Images officielles Microsoft déjà optimisées pour l'exécution |

## Cas particuliers

> [!warning] COPY . . trop tôt casse le cache
> Copier tout le code source avant d'installer les dépendances force Docker à réinstaller les dépendances à chaque changement de code, même mineur.

> [!tip] COPY plutôt qu'ADD
> `COPY` ne fait qu'une seule chose : copier des fichiers. `ADD` ajoute des comportements implicites (extraction d'archives, téléchargement d'URL) qui nuisent à la lisibilité. Préférer `COPY` sauf besoin explicite d'extraction d'archive locale.

> [!info] COPY --chown et --chmod
> `COPY --chown=<user>:<group> --chmod=<perms> src dest` attribue directement propriétaire et permissions pendant la copie, en une seule couche — voir [[Docker 08 — Sécurité des conteneurs]] pour l'intérêt par rapport à un `RUN chown` séparé. Ces deux options ne fonctionnent que sur des images Linux.

> [!info] FROM : --platform et pin par digest
> `FROM [--platform=<platform>] <image>[:<tag>|@<digest>] [AS <name>]` — `--platform` (ex. `linux/amd64`) force la plateforme cible pour une image multi-architecture ; utiliser `@sha256:...` plutôt qu'un tag fige l'image de façon immuable, contrairement à un tag qui peut être repoussé vers un contenu différent. Voir [[Docker 04 — Registry & distribution]] pour la distinction tag/digest.

> [!tip] Automatiser la mise à jour d'un digest épinglé
> Un digest figé garantit la reproductibilité mais gèle aussi les correctifs de sécurité tant que personne ne le met à jour manuellement. Des outils comme **Renovate** ou **Dependabot** détectent les nouvelles versions d'une image et ouvrent automatiquement une pull request pour mettre à jour le digest — évitant de choisir entre reproductibilité et patchs de sécurité.

> [!info] Vérifier automatiquement ces règles
> Un linter comme Hadolint détecte la plupart des écarts vus dans cette fiche (ordre du cache, `:latest`, `ADD` au lieu de `COPY`...) directement sur le Dockerfile, avant même de builder. Voir [[Docker 09 — Outils d'analyse & linting]].

> [!warning] Nettoyer apt dans la même couche, pas une couche séparée
> ```dockerfile
> # ❌ Le cache apt existe déjà dans la couche du RUN précédent : il occupe l'espace même après ce rm
> RUN apt-get update && apt-get install -y mariadb-server
> RUN rm -rf /var/lib/apt/lists/*
>
> # ✅ Même couche : le cache n'est jamais écrit "pour de vrai" dans l'image finale
> RUN apt-get update && apt-get install -y mariadb-server && \
>     rm -rf /var/lib/apt/lists/*
> ```
> Chaque couche étant immuable et additive, supprimer un fichier dans un `RUN` séparé ne réduit pas la taille de l'image : le fichier reste présent dans la couche précédente, seulement masqué. Le nettoyage doit se faire dans la **même instruction `RUN`** que l'installation.

> [!tip] --no-install-recommends sur Debian/Ubuntu
> ```dockerfile
> RUN apt-get update && apt-get install -y --no-install-recommends nginx && \
>     rm -rf /var/lib/apt/lists/*
> ```
> Par défaut, `apt-get install` installe aussi les paquets "recommandés", pas seulement les dépendances strictement nécessaires au paquet demandé. `--no-install-recommends` les exclut — un gain de taille direct, à combiner avec le nettoyage du cache apt vu ci-dessus.

> [!tip] apk --virtual pour les dépendances de build temporaires sous Alpine
> ```dockerfile
> RUN apk add --no-cache --virtual .build-deps gcc make musl-dev \
>     && make \
>     && apk del .build-deps
> ```
> `--virtual .build-deps` regroupe les paquets de compilation sous un méta-paquet nommé, supprimable en une seule commande (`apk del .build-deps`) une fois la compilation terminée. Comme pour `apt-get`, ce nettoyage ne réduit la taille de l'image que si installation et suppression se font dans la **même** instruction `RUN` — voir l'exemple à trois stages plus haut pour une solution plus robuste que `--virtual` seul.
