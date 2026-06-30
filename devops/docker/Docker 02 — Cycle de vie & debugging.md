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

## Lister et observer les conteneurs

```bash
docker ps                 # conteneurs en cours d'exécution
docker ps -a               # tous les conteneurs, y compris arrêtés
docker stats                # métriques temps réel (CPU, mémoire, réseau)
```

## Lire les logs

```bash
docker logs my-container              # logs déjà produits
docker logs -f my-container            # suivre les logs en temps réel
docker logs --tail 100 my-container    # les 100 dernières lignes
docker logs --since 10m my-container   # logs des 10 dernières minutes
```

Docker capture par défaut tout ce que le processus principal écrit sur `stdout`/`stderr` — c'est pourquoi une application bien conçue pour un conteneur écrit ses logs sur la sortie standard plutôt que dans un fichier interne.

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

## Arrêter et supprimer

| Situation | Commande |
|-----------|----------|
| Arrêt propre (signal SIGTERM, puis SIGKILL après délai) | `docker stop my-container` |
| Arrêt immédiat (signal SIGKILL direct) | `docker kill my-container` |
| Suppression d'un conteneur arrêté | `docker rm my-container` |
| Arrêt et suppression en une commande | `docker rm -f my-container` |

## Cas particuliers

> [!warning] exec ne marche que sur un conteneur démarré
> `docker exec` échoue avec une erreur explicite sur un conteneur arrêté. Toujours vérifier l'état avec `docker ps` avant de troubleshooter.

> [!tip] stop avant kill
> `docker stop` laisse le temps au processus de se terminer proprement (fermeture de connexions, sauvegarde d'état) avant de le forcer. `docker kill` ne laisse aucun délai — à réserver aux cas où le conteneur ne répond plus.

> [!info] Logs et drivers
> Le driver de logs par défaut (`json-file` ou `local`) écrit sur disque sans rotation automatique configurée par défaut — voir [[Docker — Pièges classiques]] pour le risque de saturation disque associé.
