#devops #docker #production #avancé

## Le fichier daemon.json

La plupart des réglages vus dans les fiches précédentes s'appliquent par conteneur (`docker run --memory`, `--cap-drop`...). Certains réglages, eux, s'appliquent une fois pour toutes au **démon** lui-même, via un fichier qui **n'existe pas par défaut** — à créer manuellement.

| Système | Emplacement |
|---------|-------------|
| Linux | `/etc/docker/daemon.json` |
| Windows | `C:\ProgramData\docker\config\daemon.json` |
| macOS (Docker Desktop) | `~/.docker/daemon.json` |

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<'EOF'
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "icc": false,
  "userland-proxy": false,
  "no-new-privileges": true
}
EOF
sudo systemctl restart docker
```

| Clé | Effet |
|-----|-------|
| `log-opts.max-size` / `max-file` | Rotation des logs par défaut pour **tous** les conteneurs — sans elle, un conteneur qui log abondamment peut remplir le disque (voir [[Docker — Pièges classiques]]) |
| `icc` | *Inter-container communication* — `false` désactive la communication libre entre conteneurs sur le bridge par défaut, sauf réseau explicite (voir [[Docker 06 — Réseaux]]) |
| `userland-proxy` | `false` désactive le proxy utilisateur pour la publication de ports, au profit des règles `iptables` seules — légèrement plus performant, mais peut casser certains scénarios de connexion en boucle vers `localhost` (*hairpin*) : à tester en staging avant la production |
| `no-new-privileges` | Applique par défaut à tous les conteneurs ce que `--security-opt no-new-privileges` fait individuellement (voir [[Docker 08 — Sécurité des conteneurs]]) |

> [!warning] Valider la syntaxe avant tout redémarrage
> Le format est du JSON strict : pas de virgule après le dernier élément, pas de commentaires, booléens en minuscules. Une erreur de syntaxe **empêche Docker de démarrer** — toujours valider avant d'appliquer :
> ```bash
> sudo dockerd --validate     # validation complète par le démon
> jq . /etc/docker/daemon.json   # validation JSON simple, plus rapide
> ```

## Reload à chaud vs redémarrage complet

Certaines options se rechargent sans couper les conteneurs en cours (`SIGHUP`), d'autres exigent un redémarrage complet du démon.

```bash
sudo systemctl reload docker      # ou : sudo kill -SIGHUP $(pidof dockerd)
sudo systemctl restart docker     # nécessaire pour storage-driver, data-root, log-driver...
```

| Option | Reload (`SIGHUP`) | Restart |
|--------|---------------------|---------|
| `debug`, `labels`, `live-restore`, `max-concurrent-downloads/uploads`, `shutdown-timeout` | ✅ | ✅ |
| `storage-driver`, `data-root`, `log-driver`, `default-runtime` | ❌ | ✅ |

> [!tip] En cas de doute, redémarrer reste toujours valide
> Un `restart` couvre toutes les options, y compris celles qui supporteraient un simple `reload` — la table ci-dessus sert surtout à savoir quand un `reload` (sans interruption grâce à `live-restore`, voir plus bas) suffit.

## live-restore : survivre à un redémarrage du démon

```json
{ "live-restore": true }
```

Avec `live-restore`, les conteneurs **continuent de tourner** pendant que le démon Docker redémarre (mise à jour de version, récupération après crash du démon) — sans coupure de service.

> [!warning] Limites de live-restore
> Ne s'applique pas à un arrêt manuel (`systemctl stop docker`) : dans ce cas les conteneurs s'arrêtent normalement. Les options de runtime ne doivent pas changer entre les versions de Docker pour que la reprise se passe sans accroc.

## Limites par défaut pour tous les conteneurs

```json
{
  "default-ulimits": {
    "nofile": { "Name": "nofile", "Hard": 65535, "Soft": 65535 },
    "nproc": { "Name": "nproc", "Hard": 4096, "Soft": 4096 }
  },
  "default-shm-size": "64M"
}
```

| Clé | Rôle |
|-----|------|
| `default-ulimits.nofile` | Nombre de descripteurs de fichiers ouverts simultanément par conteneur |
| `default-ulimits.nproc` | Nombre de processus autorisés par conteneur |
| `default-shm-size` | Taille de `/dev/shm` — certaines applications (navigateurs headless, certains moteurs de base de données) échouent silencieusement si ce partage mémoire est trop petit |

```bash
# Surcharger ponctuellement pour un seul conteneur
docker run --ulimit nofile=1024:2048 myapp
```

## Partition dédiée pour /var/lib/docker

Par défaut, images, conteneurs et volumes s'accumulent dans `/var/lib/docker`, sur la même partition que le reste du système. En production, une **partition dédiée** pour ce dossier évite qu'une accumulation d'images ou de logs ne sature l'espace disque utilisé par le système d'exploitation lui-même — un disque plein sur `/var/lib/docker` reste gênant, un disque plein sur `/` peut rendre la machine entière instable.

```bash
# 1. Préparer et monter la nouvelle partition
sudo mkfs.ext4 /dev/sdb1
sudo mkdir -p /mnt/docker-data
sudo mount /dev/sdb1 /mnt/docker-data
echo '/dev/sdb1 /mnt/docker-data ext4 defaults 0 2' | sudo tee -a /etc/fstab

# 2. Migrer les données existantes
sudo systemctl stop docker
sudo rsync -aP /var/lib/docker/ /mnt/docker-data/
sudo mv /var/lib/docker /var/lib/docker.bak   # conservé comme filet de sécurité, à supprimer après vérification

# 3. Pointer Docker vers le nouvel emplacement
```
```json
{ "data-root": "/mnt/docker-data" }
```
```bash
sudo systemctl start docker
docker info | grep "Docker Root Dir"   # doit afficher /mnt/docker-data
```

> [!warning] `data-root` exige un restart complet, jamais un reload
> Voir la table reload/restart plus haut — changer `data-root` sans redémarrer complètement le démon n'a aucun effet, ou pire, désynchronise l'état du démon par rapport au disque.

## Centraliser les logs au-delà de json-file

`log-driver`/`log-opts` dans `daemon.json` fixent le comportement par défaut pour tous les conteneurs (surchargeable par conteneur avec `docker run --log-driver=... --log-opt ...`, voir [[Docker 02 — Cycle de vie & debugging]]).

```json
{ "log-driver": "journald", "log-opts": { "tag": "{{.Name}}/{{.ID}}" } }
```

Avec `journald`, la rotation est gérée par systemd lui-même (plus besoin de `max-size`/`max-file`), et les logs se consultent via `journalctl CONTAINER_NAME=mon-conteneur`.

Pour un environnement avec centralisation externe des logs, Docker sait parler directement à un collecteur :

```json
// Fluentd
{ "log-driver": "fluentd", "log-opts": { "fluentd-address": "localhost:24224", "fluentd-async": "true" } }

// Splunk (HEC)
{ "log-driver": "splunk", "log-opts": { "splunk-url": "https://splunk.example.com:8088", "splunk-token": "your-hec-token" } }

// Gelf (Graylog)
{ "log-driver": "gelf", "log-opts": { "gelf-address": "udp://graylog.example.com:12201" } }
```

> [!info] Changer de log-driver exige un restart
> Comme `storage-driver` et `data-root`, `log-driver` fait partie des options qui n'acceptent pas de reload à chaud (voir la table plus haut).

## Exposer les métriques du démon (Prometheus)

```json
{ "metrics-addr": "127.0.0.1:9323", "experimental": true }
```

```bash
curl http://localhost:9323/metrics
```

Métriques notables : `engine_daemon_container_states_containers` (répartition des conteneurs par état), `engine_daemon_image_actions_seconds`, `process_cpu_seconds_total` (charge du démon lui-même).

> [!warning] Ne jamais exposer l'endpoint métriques publiquement
> `metrics-addr` sans restriction (`0.0.0.0:9323`) rend ces informations accessibles à quiconque atteint le port. Le lier à `127.0.0.1` puis passer par un reverse proxy authentifié (ou restreindre par firewall) pour tout accès distant, comme dans l'exemple ci-dessus.

## Dépannage du démon

```bash
journalctl -u docker.service -f     # logs du démon en direct
sudo dockerd --debug                 # mode debug ponctuel
```

| Erreur | Cause probable | Solution |
|--------|-------------------|----------|
| `invalid character` | Erreur de syntaxe JSON | Valider avec `jq . daemon.json` |
| `unable to configure the Docker daemon with file` | Option invalide ou mal nommée | Vérifier l'orthographe exacte de la clé dans la documentation |
| `error initializing graphdriver` | Storage driver incompatible avec les données existantes | Voir [[Docker 14 — Storage drivers (overlay2, btrfs, zfs)]] pour la procédure de migration |
| `permission denied` sur `daemon.json` | Droits de fichier incorrects | `chmod 644 /etc/docker/daemon.json` |

> [!tip] Revenir en arrière rapidement
> ```bash
> sudo systemctl stop docker
> sudo mv /etc/docker/daemon.json /etc/docker/daemon.json.bak
> sudo systemctl start docker   # repart avec la configuration par défaut
> ```

## Nettoyer les ressources inutilisées

```bash
docker system df               # voir l'espace disque utilisé par images/conteneurs/volumes/cache
docker system prune            # supprime conteneurs arrêtés, réseaux et images non référencées, cache de build
docker system prune -a         # + toutes les images non utilisées par un conteneur actif (même celles présentes localement)
```

`docker system prune` combine en une seule commande ce que feraient séparément `docker container prune`, `docker network prune` (voir [[Docker 06 — Réseaux]]) et `docker image prune` — les volumes ne sont volontairement **pas** inclus par défaut (voir [[Docker 05 — Volumes & persistance]] pour `docker volume prune`, à part).

## Cas particuliers

> [!warning] `system prune` ne touche pas aux volumes par défaut
> Contrairement à `docker compose down -v` (voir [[Docker — Pièges classiques]]), `docker system prune` ne supprime aucun volume — un ajout de `--volumes` est nécessaire pour cela, à ne jamais faire sans avoir vérifié qu'aucune donnée importante n'y est stockée.

> [!tip] Checklist minimale avant mise en production
> Partition dédiée pour `/var/lib/docker` · rotation des logs configurée (`daemon.json` ou `logging:` par service) · utilisateurs non-root · ressources limitées (`--memory`/`--cpus`) · images officielles à jour · health checks sur les services critiques · monitoring actif (`docker stats`, ou Prometheus/Grafana en externe).

## Exemple complet de daemon.json en production

Assemblage indicatif de tout ce qui précède — à adapter, pas à copier tel quel :

```json
{
  "storage-driver": "overlay2",
  "data-root": "/mnt/docker-data",
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "5", "compress": "true" },
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true,
  "default-ulimits": { "nofile": { "Name": "nofile", "Hard": 65535, "Soft": 65535 } },
  "default-shm-size": "64M",
  "registry-mirrors": ["https://mirror.gcr.io"],
  "metrics-addr": "127.0.0.1:9323"
}
```

> [!info] `docker stats` pour un contrôle ponctuel
> `docker stats` (voir [[Docker 02 — Cycle de vie & debugging]]) donne une vue immédiate de la consommation CPU/mémoire par conteneur — utile pour calibrer les valeurs de `--memory`/`--cpus` avant de les fixer définitivement.
