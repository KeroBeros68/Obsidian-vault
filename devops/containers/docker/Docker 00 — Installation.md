#devops #docker #fondamentaux #installation

## Docker Engine vs Docker Desktop

Deux produits distincts, à ne pas confondre avant de choisir une méthode d'installation.

| | Docker Engine | Docker Desktop |
|---|----------------|-----------------|
| Plateforme | Linux natif | Windows, macOS, Linux |
| Interface | CLI uniquement | GUI + CLI |
| Licence | Open source (Apache 2.0) | Gratuit usage personnel, payant en entreprise (> 250 salariés ou > 10M$ de revenus) |
| Kubernetes intégré | Non | Oui |
| Compose | Installation séparée (`docker-compose-plugin`) | Inclus |
| Virtualisation | Aucune (natif) | Oui (WSL2 sur Windows, VM sur macOS) |

> [!tip] Recommandation par contexte
> Serveur Linux → Docker Engine directement. Poste de développement Windows/macOS → Docker Desktop, ou une alternative gratuite (WSL2 sans Desktop, Colima) si la licence entreprise pose problème.

## Installer sur Linux (Ubuntu/Debian)

```bash
# 1. Désinstaller les anciennes versions (échoue silencieusement si rien n'est installé, c'est normal)
sudo apt remove docker.io docker-compose docker-doc podman-docker containerd runc

# 2. Dépendances
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# 3. Clé GPG du dépôt officiel
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 4. Ajouter le dépôt (remplacer "ubuntu" par "debian" sur Debian)
echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Installer Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 6. Vérifier
sudo docker run hello-world
```

Sur Rocky Linux/RHEL, la logique est identique mais via `dnf` : désinstaller les paquets `docker*`/`podman`/`runc` existants, ajouter le dépôt (`dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo`), puis `dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`.

> [!warning] docker.io n'est pas docker-ce
> Le paquet `docker.io` des dépôts Ubuntu/Debian est une version packagée par la distribution, souvent en retard d'une ou plusieurs versions sur `docker-ce` (le dépôt officiel Docker). Toujours désinstaller `docker.io` avant d'installer `docker-ce` pour éviter un conflit entre les deux.

## Post-installation Linux

```bash
# Utiliser Docker sans sudo : ajouter l'utilisateur au groupe docker
sudo usermod -aG docker $USER
newgrp docker   # applique le changement sans déconnexion complète

# Démarrage automatique au boot
sudo systemctl enable docker.service containerd.service
```

> [!warning] Le groupe docker équivaut à un accès root
> Appartenir au groupe `docker` permet de monter n'importe quel chemin de l'hôte dans un conteneur — c'est fonctionnellement équivalent à un accès root sur la machine, pour la même raison que l'accès au socket Docker documenté dans [[Docker 08 — Sécurité des conteneurs]]. Ne pas ajouter au groupe `docker` un utilisateur ou un service qui ne doit pas avoir ce niveau de privilège.

> [!info] Configuration du démon
> Personnaliser la rotation des logs, le storage driver ou d'autres réglages globaux passe par `/etc/docker/daemon.json`, non couvert ici en détail — voir [[Docker 10 — Configuration production & nettoyage]].

## Installer sur Windows

**Docker Desktop** (recommandé pour le développement) :

```powershell
wsl --install          # active WSL2, redémarrage possible
# Télécharger et exécuter l'installateur depuis docker.com
# Cocher "Use WSL 2 instead of Hyper-V" pendant l'installation
```

Au premier lancement, activer l'intégration WSL dans *Settings > Resources > WSL Integration* pour la ou les distributions utilisées.

**Alternative sans licence Docker Desktop** : installer Docker Engine directement dans une distribution WSL2, en suivant les étapes Ubuntu/Debian ci-dessus depuis un terminal WSL.

```bash
# Docker Engine ne démarre pas seul dans WSL2 : à ajouter dans ~/.bashrc
if ! pgrep -x "dockerd" > /dev/null; then
    sudo dockerd > /dev/null 2>&1 &
fi
```

| | Docker Desktop (Windows) | Docker Engine dans WSL2 |
|---|----------------------------|--------------------------|
| Licence | Payante au-delà de 250 salariés | Gratuite dans tous les cas |
| Interface graphique | ✅ | ❌ |
| Démarrage automatique | ✅ | Manuel (script `~/.bashrc`) |
| Kubernetes intégré | ✅ | ❌ |

## Installer sur macOS

**Docker Desktop** : télécharger le `.dmg` correspondant à l'architecture (Apple Silicon ou Intel), glisser dans *Applications*, lancer, puis ajuster CPU/mémoire/disque dans *Settings > Resources*.

**Alternative gratuite : Colima** (*Containers on Lima*) :

```bash
brew install colima docker docker-compose
colima start --cpu 4 --memory 8 --disk 60
docker run hello-world
```

> [!info] Colima n'a pas d'interface graphique
> Colima fournit le même moteur Docker que Docker Desktop, gratuitement et sans limite d'usage en entreprise, mais sans GUI ni fonctionnalités avancées (extensions, Dev Environments). Les commandes `docker`/`docker compose` restent identiques.

## Vérifier l'installation

```bash
docker version   # versions client ET serveur — si seul le client s'affiche, le daemon n'est pas accessible
docker info      # configuration complète : storage driver, root dir, nombre de conteneurs/images
docker run hello-world   # test fonctionnel de bout en bout (pull + run + affichage + arrêt)
```

## Cas particuliers

> [!warning] "Cannot connect to Docker daemon"
> Trois vérifications dans l'ordre : le service tourne-t-il (`systemctl status docker`) ? L'utilisateur est-il dans le groupe `docker` (`groups $USER`, puis reconnexion si récemment ajouté) ? Le socket existe-t-il (`ls -la /var/run/docker.sock`) ?

> [!tip] Diagnostiquer un démon qui refuse de démarrer
> `journalctl -u docker.service --no-pager -n 50` affiche les dernières lignes de log du démon — la cause (conflit de port, erreur de syntaxe dans `daemon.json`, storage driver incompatible) y est presque toujours explicite. Voir aussi [[Docker 10 — Configuration production & nettoyage]] pour les erreurs liées à `daemon.json`.

> [!info] Windows : virtualisation désactivée dans le BIOS
> Docker Desktop qui refuse de démarrer sur Windows est souvent lié à la virtualisation matérielle désactivée au niveau du BIOS/UEFI, plutôt qu'à un problème de Docker lui-même — à vérifier avant d'aller chercher plus loin.
