#devops #docker #stockage #avancé

## Le storage driver gère le système de fichiers des couches

Le **storage driver** est le mécanisme qui implémente concrètement l'empilement de couches vu dans [[Docker 01 — Images vs Conteneurs]] (union filesystem, copy-on-write) sur le système de fichiers réel de l'hôte. Le choix impacte performance et compatibilité, pas la logique des couches elle-même.

```bash
docker info | grep "Storage Driver"
# Storage Driver: overlay2
```

## Choisir le bon driver

| Driver | Kernel requis | Performance | Statut |
|--------|-----------------|--------------|--------|
| **overlay2** | 4.0+ | Excellente | ✅ Recommandé — défaut depuis Docker 18.09 |
| **btrfs** | Système `btrfs` | Bonne | Adapté si l'hôte utilise déjà btrfs |
| **zfs** | Système ZFS | Bonne | Adapté si l'hôte utilise déjà ZFS |
| **devicemapper** | 2.6.9+ | Moyenne | ⚠️ Déprécié |
| **vfs** | Tout | Très mauvaise | Debug uniquement (aucun partage de couches) |
| **aufs** | Patch noyau spécifique | Bonne | ❌ Obsolète |

`overlay2` s'appuie sur **OverlayFS**, intégré au noyau Linux depuis longtemps, avec un mécanisme copy-on-write natif — c'est le choix par défaut pour toute installation moderne, sauf contrainte technique précise (hôte déjà construit sur btrfs ou ZFS).

## Configurer overlay2

```json
{
  "storage-driver": "overlay2",
  "storage-opts": ["overlay2.override_kernel_check=true"]
}
```

| Option | Rôle |
|--------|------|
| `overlay2.override_kernel_check` | Ignore la vérification de version de noyau (utile sur un noyau récent mal détecté) |
| `overlay2.size` | Limite la taille utilisable par conteneur (nécessite un filesystem avec quotas activés) |

## Changer de storage driver : une opération destructive

> [!warning] Changer de driver supprime toutes les images et conteneurs existants
> Les données sous `/var/lib/docker` sont structurées différemment selon le driver actif — il n'existe pas de conversion à la volée. Changer `storage-driver` dans `daemon.json` (voir [[Docker 10 — Configuration production & nettoyage]], qui exige un restart complet, jamais un reload) revient à repartir d'un `/var/lib/docker` vide.

Procédure de migration :

```bash
# 1. Sauvegarder les images importantes (l'historique de build n'est pas conservé, seule l'image l'est)
docker save myapp:latest > myapp.tar

# 2. Arrêter Docker et vider les données de l'ancien driver
sudo systemctl stop docker
sudo rm -rf /var/lib/docker

# 3. Changer le storage-driver dans daemon.json, puis redémarrer
sudo systemctl start docker

# 4. Recharger les images sauvegardées
docker load < myapp.tar
```

> [!tip] Un `docker save`/`docker load` par image, pas un simple rsync
> Contrairement à `data-root` (voir [[Docker 10 — Configuration production & nettoyage]]), où `rsync` suffit à déplacer les données telles quelles, un changement de **driver** modifie leur structure interne — seul un export/import via `docker save`/`docker load` (ou un `docker push` vers un registry avant la bascule) préserve les images à travers la migration.

## Cas particuliers

> [!info] `vfs` n'est pas un driver de production
> `vfs` copie intégralement chaque couche sans aucun partage ni copy-on-write — extrêmement lent et gourmand en espace disque. Sa seule utilité réelle est de fournir un comportement de référence simple pour déboguer un problème qui pourrait venir d'un autre driver.

> [!warning] `error initializing graphdriver` au démarrage
> Ce message dans les logs du démon (voir [[Docker 10 — Configuration production & nettoyage]]) signale une incompatibilité entre le `storage-driver` configuré et les données déjà présentes sous `data-root` — généralement après un changement de driver sans avoir vidé l'ancien répertoire au préalable.
