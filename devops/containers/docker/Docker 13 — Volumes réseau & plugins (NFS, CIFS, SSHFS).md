#devops #docker #volumes #avancé

## Quand le stockage local ne suffit plus

[[Docker 05 — Volumes & persistance]] couvre le stockage **local** : volumes standards, bind mounts, tmpfs — tous stockés sur le disque de la machine qui exécute le conteneur. Dès qu'un cluster comporte plusieurs hôtes Docker (Swarm, standalone multi-serveurs), un conteneur qui migre d'une machine à l'autre repart à vide s'il ne dépend que du stockage local : les données restent sur l'ancien hôte.

Le driver `local` de Docker supporte nativement deux protocoles réseau (**NFS**, **CIFS/SMB**) sans plugin tiers, et des plugins tiers étendent le champ à d'autres backends (SSHFS, cloud).

## Où sont réellement stockées les données

```bash
docker volume inspect mon-volume
# "Mountpoint": "/var/lib/docker/volumes/mon-volume/_data"
```

| Type de montage | Emplacement physique | Géré par |
|-------------------|--------------------------|-----------|
| Volume standard | `/var/lib/docker/volumes/<nom>/_data` | Démon Docker |
| Bind mount | N'importe où sur l'hôte | Vous |
| tmpfs | RAM du système | Le système d'exploitation |
| Volume réseau | Serveur distant (NFS, CIFS, cloud) | Driver `local` (avec options) ou plugin |

> [!info] Docker Desktop : les volumes ne sont pas visibles directement
> Sur macOS/Windows, Docker Desktop exécute le démon dans une VM légère — le chemin `/var/lib/docker/volumes/` existe dans cette VM, pas directement accessible depuis l'explorateur de fichiers de l'hôte, contrairement à un hôte Linux natif.

## Volumes NFS

NFS est le protocole standard de partage de fichiers entre systèmes Linux.

```bash
# Prérequis : client NFS installé sur l'hôte Docker
sudo apt install nfs-common        # Debian/Ubuntu
sudo dnf install nfs-utils         # RHEL/Rocky/AlmaLinux

# Création du volume
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.50,nolock,soft,rw \
  --opt device=:/exports/data \
  mon-volume-nfs

docker run -v mon-volume-nfs:/data nginx
```

| Option | Rôle |
|--------|------|
| `addr` | Adresse IP ou hostname du serveur NFS |
| `nolock` | Désactive le verrouillage de fichiers (utile en NFSv3) |
| `soft` | Retourne une erreur si le serveur ne répond pas, au lieu d'attendre indéfiniment (`hard`) |
| `rw` | Montage en lecture-écriture |
| `nfsvers=4` | Force une version précise du protocole NFS |

> [!tip] `soft` plutôt que `hard` en production
> Avec `hard` (comportement par défaut de NFS), une panne du serveur NFS bloque indéfiniment tout processus qui tente d'accéder au volume. `soft` retourne une erreur exploitable à la place — au prix d'un risque de corruption si une écriture est interrompue, un compromis généralement acceptable face au risque de blocage total.

## Volumes CIFS/SMB

CIFS (SMB) est le protocole de partage de fichiers de Windows — utile pour intégrer Docker à un serveur Windows ou un NAS grand public (Synology, QNAP).

```bash
sudo apt install cifs-utils   # ou dnf install cifs-utils

docker volume create --driver local \
  --opt type=cifs \
  --opt o=addr=192.168.1.100,username=utilisateur,password=motdepasse,vers=3.0 \
  --opt device=//192.168.1.100/partage \
  mon-volume-cifs
```

> [!warning] Le mot de passe apparaît en clair dans `docker volume inspect`
> Passer `password=...` directement dans les options du volume l'expose à quiconque peut exécuter `docker volume inspect`. Préférer un fichier d'identifiants dédié :
> ```bash
> printf 'username=utilisateur\npassword=motdepasse\n' > /etc/docker-cifs-credentials
> chmod 600 /etc/docker-cifs-credentials
>
> docker volume create --driver local \
>   --opt type=cifs \
>   --opt o=credentials=/etc/docker-cifs-credentials,vers=3.0 \
>   --opt device=//serveur/partage \
>   mon-volume-cifs-secure
> ```

## NFS vs CIFS : lequel choisir

| Critère | NFS | CIFS/SMB |
|---------|-----|----------|
| Environnement | Linux vers Linux | Windows, Samba, NAS grand public |
| Performance | Meilleure | Légèrement inférieure |
| Authentification | Par IP/réseau | Par utilisateur/mot de passe |
| Cas d'usage | Clusters Linux, HPC | Partages Windows, PME avec NAS |

## Plugins de volumes : l'exemple SSHFS

Un plugin de volume étend Docker à des backends de stockage non supportés nativement. **SSHFS** (monter un système de fichiers distant via SSH) illustre le principe :

```bash
# Installer le plugin (demande une confirmation de privilèges)
docker plugin install vieux/sshfs

# Vérifier qu'il est actif
docker plugin ls
# ID             NAME                  ENABLED
# 3cbdf63cd7cb   vieux/sshfs:latest    true

# Créer un volume qui monte un chemin distant via SSH
docker volume create --driver vieux/sshfs \
  -o sshcmd=user@remote-host:/path/on/remote \
  mon-volume-sshfs

docker run -v mon-volume-sshfs:/data ...
```

Le transfert passe entièrement par SSH (authentification et chiffrement déjà éprouvés), sans configuration réseau supplémentaire côté serveur distant — au prix des performances plus limitées d'un accès réseau par rapport à un stockage local ou NFS.

## Cas particuliers

> [!warning] Volumes réseau : tester la latence avant la production
> Un volume NFS/CIFS/SSHFS ajoute une dépendance réseau à chaque accès disque. Une latence réseau qui semblait négligeable en test peut devenir un goulot d'étranglement notable pour une base de données à fort trafic d'écriture — mesurer avant de généraliser à un service critique.

> [!tip] Isoler l'accès réseau des volumes
> Restreindre l'accès au serveur NFS/CIFS/SSHFS par IP ou VLAN dédié plutôt que de l'exposer largement — un volume réseau mal isolé étend la surface d'attaque au-delà du seul hôte Docker.

## Pour aller plus loin

Sauvegarde/restauration (pattern `tar`, outils Restic/Rclone/Offen) et labels de volumes restent décrits dans [[Docker 05 — Volumes & persistance]], quel que soit le driver de stockage utilisé.
