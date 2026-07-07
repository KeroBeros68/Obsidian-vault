#devops #docker #fondamentaux #debugging

## Les états d'un conteneur

Un conteneur traverse plusieurs états entre sa création et sa suppression. Comprendre ces états est la base de tout diagnostic.

```
created → running → (paused) → stopped → removed
            ↑___________________|
              restart possible
```

| État | Signification |
|------|---------------|
| **created** | Conteneur créé mais jamais démarré |
| **running** | Processus principal actif |
| **paused** | Processus suspendus (gelés), mémoire conservée |
| **stopped (exited)** | Processus terminé, conteneur toujours présent sur disque |
| **removed** | Conteneur supprimé définitivement (`docker rm`) |

## Décoder les exit codes

Quand un conteneur s'arrête, Docker conserve son **exit code** — visible tant que le conteneur n'est pas supprimé (`docker rm`).

| Exit code | Signification | Cause probable |
|-----------|----------------|------------------|
| `0` | Succès | Le processus s'est terminé normalement |
| `1` | Erreur générique | Bug applicatif, mauvaise configuration |
| `126` | Permission denied | Le binaire n'est pas exécutable |
| `127` | Command not found | La commande n'existe pas dans l'image |
| `137` | SIGKILL (128+9) | OOM kill ou `docker kill` |
| `139` | SIGSEGV (128+11) | Segmentation fault (crash mémoire) |
| `143` | SIGTERM (128+15) | Arrêt propre via `docker stop` |

> [!tip] La règle des 128+
> Un exit code supérieur à 128 signifie que le processus a été tué par un signal Linux : `code - 128 = numéro du signal`. `137 = 128 + 9` (SIGKILL), `143 = 128 + 15` (SIGTERM). `143` est donc un arrêt normal, `137` mérite une investigation (voir plus bas).

```bash
docker ps -a --filter "exited=137"                         # tous les conteneurs tués par SIGKILL/OOM
docker ps -a --filter "status=exited" --filter "exited=1"  # tous les conteneurs en erreur
```

## Lister et observer les conteneurs

```bash
docker ps                 # conteneurs en cours d'exécution
docker ps -a               # tous les conteneurs, y compris arrêtés
docker stats                # métriques temps réel (CPU, mémoire, réseau)
docker stats --no-stream mon-conteneur   # une seule mesure, sans flux continu
```

Colonnes à surveiller dans `docker stats` : `MEM USAGE / LIMIT` (proche de la limite = risque d'OOM imminent), `CPU %`, `NET I/O`.

## Lire les logs

```bash
docker logs my-container              # logs déjà produits
docker logs -f my-container            # suivre les logs en temps réel
docker logs --tail 100 my-container    # les 100 dernières lignes
docker logs --since 10m my-container   # logs des 10 dernières minutes
docker logs -t my-container            # avec horodatage de chaque ligne
```

Docker capture par défaut tout ce que le processus principal écrit sur `stdout`/`stderr` — c'est pourquoi une application bien conçue pour un conteneur écrit ses logs sur la sortie standard plutôt que dans un fichier interne.

> [!warning] `docker logs` vide malgré une application active
> `docker logs` ne fonctionne que si le driver de log est `json-file` (par défaut) ou `local`. Avec un driver `syslog`, `journald` ou `fluentd` configuré (voir [[Docker 10 — Configuration production & nettoyage]]), les logs ne transitent plus par cette commande. Vérifier le driver actif avec `docker inspect --format '{{.HostConfig.LogConfig.Type}}' my-container` — si c'est `journald`, chercher plutôt via `journalctl CONTAINER_NAME=my-container`.

## Entrer dans un conteneur en cours d'exécution

```bash
docker exec -it my-container bash    # ouvre un shell interactif
docker exec my-container ls /app      # exécute une commande ponctuelle
docker exec -u root my-container whoami   # exécute en tant que root
```

`docker exec` ne fonctionne que sur un conteneur **déjà démarré** — contrairement à `docker run`, qui en crée un nouveau. Confusion fréquente chez les débutants.

## Inspecter la configuration

```bash
docker inspect my-container
docker inspect --format '{{.State.Status}}' my-container
docker inspect --format '{{.Config.Image}}' my-container
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container
```

`docker inspect` retourne un objet JSON complet (configuration, état, réseau, montages). Le flag `--format` permet d'en extraire un champ précis avec la syntaxe Go template.

### Champs à vérifier en priorité pour un diagnostic

| Champ | Ce qu'il révèle |
|-------|-------------------|
| `.State.ExitCode` | Le code de sortie (voir la table plus haut) |
| `.State.OOMKilled` | `true` si le conteneur a été tué par manque de mémoire |
| `.State.Error` | Message d'erreur Docker, souvent révélateur (ex. échec de démarrage) |
| `.RestartCount` | Nombre de redémarrages déjà effectués — élevé = boucle de crash |
| `.Config.Entrypoint` / `.Config.Cmd` | Ce que le conteneur essaie réellement de lancer |
| `.HostConfig.Memory` | Limite mémoire configurée en octets (`0` = illimitée) |
| `.Mounts` | Volumes et bind mounts montés (utile pour un problème de permissions) |

```bash
# Plusieurs champs en une seule commande, pratique pour un premier coup d'œil
docker inspect --format 'Exit={{.State.ExitCode}} OOM={{.State.OOMKilled}} Restarts={{.RestartCount}}' my-container
```

## Arrêter et supprimer

| Situation | Commande |
|-----------|----------|
| Arrêt propre (signal SIGTERM, puis SIGKILL après délai) | `docker stop my-container` |
| Arrêt propre avec délai personnalisé (30s au lieu de 10s) | `docker stop -t 30 my-container` |
| Arrêt immédiat (signal SIGKILL direct) | `docker kill my-container` |
| Suppression d'un conteneur arrêté | `docker rm my-container` |
| Arrêt et suppression en une commande | `docker rm -f my-container` |
| Modifier une politique de redémarrage sans recréer le conteneur | `docker update --restart unless-stopped my-container` |

> [!warning] `docker stop` bloqué : un problème de PID 1
> Si le processus principal (PID 1) est un script shell qui lance l'application sans `exec` (ex. `./mon_app` au lieu de `exec ./mon_app`), le `SIGTERM` de `docker stop` est reçu par le shell, qui l'ignore souvent — l'arrêt traîne jusqu'au `SIGKILL` de secours. Deux solutions : corriger le script avec `exec` (voir l'entrypoint dans [[Docker 03 — Dockerfile]]), ou lancer le conteneur avec `docker run --init`, qui insère un mini-init capable de relayer correctement les signaux même à un script imparfait.

## Limiter les ressources d'un conteneur

```bash
docker run -d \
  --memory="512m" \
  --memory-swap="1g" \
  --cpus="1.5" \
  --pids-limit=100 \
  --restart=unless-stopped \
  myapp:latest
```

| Flag | Effet |
|------|-------|
| `--memory` | Mémoire RAM maximale ; au-delà, le noyau **OOM-kill** le processus principal du conteneur |
| `--memory-swap` | Total mémoire + swap autorisé (doit être ≥ `--memory` pour autoriser du swap) |
| `--pids-limit` | Nombre maximal de processus que le conteneur peut créer — protection contre un fork bomb, accidentel ou malveillant |
| `--cpus` | Nombre de cœurs CPU équivalents alloués (`1.5` = une cœur et demi, pas un cœur dédié) |

Sans limite, un seul conteneur peut consommer toute la RAM ou tout le CPU de l'hôte et affecter les autres conteneurs qui y tournent — indispensable dès qu'une machine héberge plusieurs services.

## Surveiller les événements en temps réel

```bash
docker events                              # flux temps réel de tous les événements
docker events --filter "type=container"    # uniquement les événements conteneurs
docker events --filter "event=start"       # uniquement les démarrages
docker events --since '2026-07-05T12:00:00' --until '2026-07-05T13:00:00'   # fenêtre passée
```

`docker events` diffuse en direct les créations, démarrages, arrêts et suppressions de conteneurs, images, volumes et réseaux — utile pour observer *quand* et *pourquoi* un conteneur redémarre en boucle, sans avoir à interroger `docker ps` en continu.

## Cas particuliers

> [!warning] Un conteneur OOM-killed redémarre silencieusement selon la politique restart
> Si `--memory` est trop bas pour l'application, le noyau tue le processus principal (code de sortie `137`) dès qu'il dépasse la limite. Avec `restart: unless-stopped` (voir [[Docker 07 — Docker Compose]]), le conteneur repart aussitôt et peut entrer dans une boucle de crash silencieuse si la cause n'est pas corrigée. `docker events --filter "event=oom"` permet de confirmer que c'est bien la cause.

> [!tip] Confirmer un OOM kill à la source
> `docker inspect --format '{{.State.OOMKilled}}' my-container` confirme si Docker a tué le conteneur pour dépassement de `--memory` (`true`/`false`). Si `false` alors que le conteneur meurt quand même en `137`, le manque de mémoire peut venir de l'**hôte** lui-même plutôt que de la limite du conteneur — dans ce cas, chercher côté noyau : `sudo dmesg | grep -i "killed process"` ou `sudo journalctl -k | grep -i oom`.

> [!info] Débugger une image sans shell (distroless, scratch)
> `docker exec -it ... sh` échoue sur les images minimales sans shell (voir [[Docker — Pièges classiques]]). Docker Desktop (licence Pro/Team/Business) propose `docker debug my-container`, qui attache un shell de diagnostic externe (avec `curl`, `vim`, `strace`...) à n'importe quel conteneur ou image, y compris arrêté — sans modifier l'image elle-même. Alternative gratuite pour le débogage réseau : un conteneur `nicolaka/netshoot` lancé sur le même réseau (voir [[Docker 06 — Réseaux]]).

> [!warning] exec ne marche que sur un conteneur démarré
> `docker exec` échoue avec une erreur explicite sur un conteneur arrêté. Toujours vérifier l'état avec `docker ps` avant de troubleshooter.

> [!tip] stop avant kill
> `docker stop` laisse le temps au processus de se terminer proprement (fermeture de connexions, sauvegarde d'état) avant de le forcer. `docker kill` ne laisse aucun délai — à réserver aux cas où le conteneur ne répond plus.

> [!info] Logs et drivers
> Le driver de logs par défaut (`json-file` ou `local`) écrit sur disque sans rotation automatique configurée par défaut — voir [[Docker — Pièges classiques]] pour le risque de saturation disque associé.
