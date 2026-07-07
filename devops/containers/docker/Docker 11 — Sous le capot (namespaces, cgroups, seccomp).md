#devops #docker #avancé #sécurité

## Les 4 piliers de l'isolation

Quand `docker run nginx` s'exécute, Docker ne crée pas une machine virtuelle : il demande au noyau Linux d'isoler et de limiter un processus via quatre mécanismes complémentaires.

| Pilier | Question | Mécanisme |
|--------|----------|-----------|
| **Namespaces** | Que *voit* le conteneur ? | Isolation de la vue système (processus, réseau, fichiers...) |
| **Cgroups** | Combien le conteneur *utilise* ? | Limitation des ressources (CPU, mémoire, I/O) — voir [[Docker 02 — Cycle de vie & debugging]] |
| **Capabilities** | Que *peut faire* le conteneur ? | Fragmentation des privilèges root — voir [[Docker 08 — Sécurité des conteneurs]] |
| **Seccomp** | Quels appels système sont *autorisés* ? | Filtrage au niveau noyau |

Cette fiche détaille les mécanismes eux-mêmes et comment les inspecter ; les fiches liées ci-dessus couvrent leur usage pratique au quotidien.

## Namespaces : ce que le conteneur voit

| Namespace | Isole | Exemple concret |
|-----------|-------|-------------------|
| **PID** | Les processus | Le conteneur voit son processus principal comme PID 1 |
| **NET** | Le réseau | Pile réseau, IP, ports propres (voir [[Docker 06 — Réseaux]]) |
| **MNT** | Les montages | Système de fichiers racine propre |
| **UTS** | Le hostname | Le conteneur peut avoir son propre nom d'hôte |
| **IPC** | La communication inter-processus | Files de messages isolées |
| **USER** | Les utilisateurs | Le root du conteneur peut être mappé sur un UID non-root de l'hôte |
| **CGROUP** | La vue des cgroups | Depuis Linux 4.6+ |

```bash
# Sur l'hôte : le vrai PID du processus, tel que vu par le noyau
docker inspect --format '{{.State.Pid}}' mon-conteneur    # ex: 12345

# Dans le conteneur : le même processus se voit comme PID 1
docker exec mon-conteneur ps aux
# USER  PID  COMMAND
# root  1    nginx

# Lister les namespaces réellement utilisés par ce processus
PID=$(docker inspect --format '{{.State.Pid}}' mon-conteneur)
ls -la /proc/$PID/ns/
```

Deux processus qui partagent le même identifiant de namespace (entre crochets, ex. `pid:[4026532xxx]`) partagent la même vue — c'est ce que fait par exemple `docker run --network container:autre-conteneur` (voir [[Docker 06 — Réseaux]]) en partageant explicitement le namespace réseau.

> [!info] PID 1 et signaux
> Le comportement particulier de PID 1 vis-à-vis des signaux (`SIGTERM` pas toujours transmis à un script sans `exec`) est détaillé dans [[Docker 02 — Cycle de vie & debugging]] (`docker stop` bloqué, flag `--init`).

## User namespaces et mode rootless

Le **user namespace** mappe l'UID 0 (root) *dans* le conteneur vers un utilisateur non privilégié *sur* l'hôte — un mécanisme distinct de l'utilisateur non-root applicatif (voir [[Docker 08 — Sécurité des conteneurs]]), puisqu'il protège même si le processus obtient un accès root à l'intérieur du conteneur.

| Mode | Démon Docker | Conteneurs | Cas d'usage |
|------|----------------|-------------|-------------|
| `userns-remap` | Root | UID remappé | Durcir des conteneurs existants sans tout changer |
| Rootless | Non-root | UID remappé | Réduire la surface d'attaque du démon lui-même |

```bash
# Vérifier si le démon utilise les user namespaces
docker info | grep -i userns
```

### Installer le mode rootless

```bash
# Prérequis : kernel 5.11+ (ou 4.18+ configuré), newuidmap/newgidmap, délégation cgroup v2
dockerd-rootless-setuptool.sh check
sudo apt install uidmap dbus-user-session

# En tant qu'utilisateur normal, PAS root
dockerd-rootless-setuptool.sh install
```

```bash
# ~/.bashrc — pointer la CLI docker vers le démon rootless de cet utilisateur
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock
```

```bash
# Démarrage automatique au boot, sans session interactive ouverte
systemctl --user enable docker
loginctl enable-linger $(whoami)

docker run --rm hello-world   # vérifier que le démon rootless répond
```

| Limitation | Alternative |
|--------------|--------------|
| Ports < 1024 inaccessibles | Port > 1024 + reverse proxy |
| Réseau overlay limité | VPN ou réseau externe |
| Cgroup v1 non supporté | Migrer vers cgroup v2 |
| AppArmor/SELinux limités | Profils utilisateur, plutôt que système |
| `--privileged` indisponible | Non nécessaire — c'est précisément l'objectif du mode rootless |

> [!tip] Rootless convient au dev et au CI, pas forcément à la prod critique
> Environnements de développement, CI/CD multi-tenant, ou workloads sans besoin de ports privilégiés sont les cas d'usage les plus naturels. Pour une production critique nécessitant à la fois rootless et orchestration, combiner avec un runtime dédié (Kata Containers) plutôt que d'espérer que le mode rootless seul couvre tous les besoins.

### Configurer userns-remap (alternative : démon root, conteneurs remappés)

Contrairement au mode rootless (démon lui-même non-root), `userns-remap` garde un démon root mais mappe les UID de **tous les conteneurs** vers une plage non privilégiée :

```bash
# 1. Utilisateur dédié au remapping
sudo useradd -r -s /bin/false dockremap

# 2. Plage d'UID/GID déléguée à cet utilisateur
echo "dockremap:100000:65536" | sudo tee -a /etc/subuid
echo "dockremap:100000:65536" | sudo tee -a /etc/subgid
```

```json
{ "userns-remap": "dockremap" }
```

```bash
sudo systemctl restart docker

# Vérifier : le PID du conteneur doit apparaître avec l'UID mappé (100000), pas 0, côté hôte
docker run -d --name test alpine sleep 3600
ps aux | grep "sleep 3600"
```

> [!warning] Volumes et permissions avec userns-remap
> Les fichiers créés dans un volume appartiennent, côté hôte, à l'UID mappé (ex. `100000`) plutôt qu'à l'UID apparent dans le conteneur. Un conteneur qui doit accéder à des fichiers existants avec leur propriétaire d'origine peut nécessiter `--userns=host` pour ce cas précis, au prix de perdre la protection du remapping pour ce conteneur.

## Cgroups v1 vs v2

La v2 (hiérarchie unifiée, meilleur accounting) est désormais le standard recommandé — Kubernetes a placé le support de la v1 en mode maintenance depuis la version 1.31 (août 2024).

```bash
# Quelle version est active sur cet hôte ?
mount | grep cgroup
# cgroup2 on /sys/fs/cgroup type cgroup2   → v2
# cgroup on /sys/fs/cgroup/cpu type cgroup → v1

# Ou directement via Docker
docker info | grep -i cgroup
# Cgroup Driver: systemd
# Cgroup Version: 2
```

```bash
# Lire la limite mémoire réelle appliquée à un conteneur (driver systemd)
PID=$(docker inspect --format '{{.State.Pid}}' mon-conteneur)
cat /proc/$PID/cgroup   # localise le chemin exact, qui dépend du driver (systemd vs cgroupfs)

CONTAINER_ID=$(docker inspect --format '{{.Id}}' mon-conteneur)
cat /sys/fs/cgroup/system.slice/docker-$CONTAINER_ID.scope/memory.max
cat /sys/fs/cgroup/system.slice/docker-$CONTAINER_ID.scope/memory.current
```

> [!info] Limiter aussi le nombre de processus
> En complément de `--memory`/`--cpus` (voir [[Docker 02 — Cycle de vie & debugging]]), `--pids-limit=100` borne le nombre de processus qu'un conteneur peut créer — utile contre un fork bomb, accidentel ou malveillant.

## Capabilities : le détail derrière --cap-drop

Docker retire par défaut la plupart des capacités dangereuses mais en conserve typiquement 14, suffisantes pour la majorité des usages courants (liste indicative, peut varier selon la version de Docker) :

`CAP_CHOWN`, `CAP_DAC_OVERRIDE`, `CAP_FOWNER`, `CAP_FSETID`, `CAP_KILL`, `CAP_SETGID`, `CAP_SETUID`, `CAP_SETPCAP`, `CAP_NET_BIND_SERVICE`, `CAP_NET_RAW`, `CAP_SYS_CHROOT`, `CAP_MKNOD`, `CAP_AUDIT_WRITE`, `CAP_SETFCAP`.

Parmi les capacités **retirées** par défaut, les plus critiques :

| Capacité retirée | Ce qu'elle permettrait | Risque |
|--------------------|---------------------------|--------|
| `CAP_SYS_ADMIN` | Presque tout (mount, namespaces, ptrace...) | Critique — souvent qualifiée de "nouveau root" |
| `CAP_SYS_MODULE` | Charger des modules noyau | Critique |
| `CAP_NET_ADMIN` | Configurer le réseau (iptables, interfaces) | Élevé |
| `CAP_SYS_PTRACE` | Déboguer d'autres processus | Élevé |

```bash
# Voir les capacités effectives d'un conteneur
docker run --rm alpine cat /proc/1/status | grep Cap
# CapEff: 00000000a80425fb

# Décoder ce masque (nécessite libcap-ng-utils sur l'hôte)
capsh --decode=00000000a80425fb
```

Voir [[Docker 08 — Sécurité des conteneurs]] pour l'usage pratique de `--cap-drop`/`--cap-add` au quotidien.

## Seccomp : filtrer les appels système

**Seccomp** (*Secure Computing Mode*) filtre les appels système qu'un processus peut effectuer — un quatrième pilier indépendant des capacités : même avec la bonne capacité, un appel système bloqué par seccomp échoue quand même avec `EPERM`/`EACCES`.

Docker applique par défaut un profil qui bloque plusieurs dizaines de syscalls jugés à risque, sans affecter la compatibilité de la plupart des applications :

| Syscall bloqué | Effet potentiel | Pourquoi bloqué |
|-----------------|--------------------|--------------------|
| `reboot` | Redémarrer le système | Impact sur l'hôte entier |
| `mount` / `umount` | Monter/démonter des filesystems | Vecteur d'évasion de conteneur |
| `ptrace` | Déboguer d'autres processus | Injection de code |
| `kexec_load` | Charger un nouveau noyau | Prise de contrôle totale |
| `bpf` | Programmes noyau eBPF | Surveillance/modification du noyau |
| `pivot_root` | Changer le filesystem racine | Vecteur d'évasion de conteneur |

```bash
# Voir le profil de sécurité appliqué (seccomp, AppArmor...)
docker inspect --format '{{.HostConfig.SecurityOpt}}' mon-conteneur

# Désactiver seccomp (dangereux — uniquement pour un diagnostic ponctuel)
docker run --security-opt seccomp=unconfined alpine

# Appliquer un profil personnalisé plus restrictif
docker run --security-opt seccomp=/path/to/profile.json alpine
```

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    { "names": ["read", "write", "exit", "exit_group"], "action": "SCMP_ACT_ALLOW" }
  ]
}
```

Un profil seccomp personnalisé liste explicitement les syscalls autorisés (`SCMP_ACT_ALLOW`), tout le reste étant refusé par défaut (`SCMP_ACT_ERRNO`) — le profil par défaut de Docker (disponible sur le dépôt `moby/moby`) sert de base de départ réaliste plutôt que de repartir de zéro.

> [!warning] `seccomp=unconfined` ne doit jamais atteindre la production
> Désactiver seccomp retire une couche de protection contre l'évasion de conteneur, même avec des capacités et un utilisateur non-root déjà restreints. À réserver strictement à un diagnostic ponctuel, jamais comme solution à un problème de compatibilité en production — un profil personnalisé ciblé est toujours préférable.

## AppArmor / SELinux : vérifier ce qui est actif

En complément des exemples d'activation dans [[Docker 08 — Sécurité des conteneurs]], vérifier concrètement ce qui est appliqué :

```bash
# AppArmor est-il actif sur l'hôte ?
cat /sys/module/apparmor/parameters/enabled   # Y = actif

# Quel profil AppArmor un conteneur donné utilise-t-il ?
docker inspect --format '{{.AppArmorProfile}}' mon-conteneur
```

## Conteneur vs machine virtuelle : où se situe la frontière

| Aspect | Machine virtuelle | Conteneur |
|--------|---------------------|-----------|
| Isolation | OS invité complet + hyperviseur | Namespaces + cgroups |
| Noyau | Séparé (chaque VM a le sien) | Partagé avec l'hôte |
| Démarrage | Secondes à dizaines de secondes | Centaines de ms à quelques secondes |
| Sécurité | Frontière matérielle, forte | Noyau partagé, plus faible par nature |

> [!warning] Un conteneur n'est pas un sandbox parfait
> Une vulnérabilité dans le noyau Linux expose potentiellement **tous** les conteneurs de la machine, puisqu'ils le partagent — contrairement à une VM, où une faille reste cantonnée à l'hyperviseur ou à l'invité concerné. C'est précisément pour compenser cette différence que namespaces, cgroups, capabilities et seccomp se combinent : aucun des quatre pris isolément ne suffit.

## Checklist de durcissement (priorité)

1. `--cap-drop=ALL` puis `--cap-add` minimal (voir [[Docker 08 — Sécurité des conteneurs]])
2. `--read-only` + `--tmpfs /tmp`
3. `--security-opt no-new-privileges`
4. User namespaces / mode rootless quand c'est possible
5. Profil seccomp par défaut conservé (voire personnalisé si besoin)
6. Limites cgroups : `--memory`, `--pids-limit`, `--cpus`
7. LSM (AppArmor/SELinux) comme couche supplémentaire
