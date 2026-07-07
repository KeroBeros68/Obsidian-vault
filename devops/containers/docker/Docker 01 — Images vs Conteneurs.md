#devops #docker #fondamentaux

## Images vs Conteneurs

Une **image** est un modèle figé (lecture seule) ; un **conteneur** est une instance en cours d'exécution de cette image, à laquelle Docker ajoute une couche modifiable.

## Couches et union filesystem

Une image Docker n'est pas un bloc monolithique : elle est composée de **couches** (*layers*) empilées par un filesystem en union (driver `overlay2` sous Linux).

- Chaque couche correspond à un ensemble de changements (ajout, modification, suppression de fichiers).
- Les couches sont **immuables** et **partagées** entre images : si dix images utilisent la même base `ubuntu`, cette couche n'est stockée qu'une seule fois sur le disque.
- Au sommet des couches d'image (toutes en lecture seule), Docker ajoute une **couche d'écriture** propre à chaque conteneur lancé.

```
image (lecture seule)          conteneur (lecture-écriture)
┌─────────────────────┐
│ Layer 3 — app code   │   →   ┌─────────────────────────┐
│ Layer 2 — dépendances │   →   │ Couche RW du conteneur   │
│ Layer 1 — OS de base  │   →   └─────────────────────────┘
└─────────────────────┘
```

> [!info] Toutes les instructions Dockerfile ne créent pas de couche
> Seules `FROM`, `RUN`, `COPY` et `ADD` produisent une nouvelle couche de fichiers. Les autres instructions (`ENV`, `LABEL`, `EXPOSE`, `CMD`, `ENTRYPOINT`, `USER`, `WORKDIR`...) ne modifient que les **métadonnées** de l'image (stockées dans son fichier de configuration), sans ajouter de données au système de fichiers — voir [[Docker 03 — Dockerfile]] pour le détail de chaque instruction.

## Content-Addressable Storage : l'identification par hash

Chaque couche est identifiée par le **hash SHA256 de son contenu**, pas par son nom ou sa position dans le Dockerfile. C'est ce mécanisme précis — appelé *Content-Addressable Storage* (CAS) — qui permet la déduplication mentionnée plus haut : si deux images partagent une couche dont le contenu produit le même hash (ex. la même base `alpine:3.19`), Docker ne la stocke qu'une seule fois sur le disque, qu'elle soit locale ou sur un registry.

Conséquence pratique lors d'un `docker pull` : seules les couches dont le hash n'est pas déjà présent localement sont téléchargées — les couches partagées avec une image déjà présente sont réutilisées telles quelles.

## Copy-on-write : pourquoi c'est rapide

Le conteneur ne copie **rien** au démarrage — il lit directement dans les couches de l'image.

- **Lecture** d'un fichier présent dans une couche d'image → servi directement, aucune copie.
- **Écriture** dans un fichier d'une couche d'image → le fichier est d'abord copié vers la couche du conteneur (*copy-on-write*), puis modifié là.
- **Suppression** d'un fichier d'une couche inférieure → un fichier "whiteout" est créé dans la couche du conteneur pour masquer le fichier, sans toucher à la couche d'origine.

Cette stratégie explique pourquoi démarrer un conteneur prend quelques millisecondes : aucun gros transfert de données, juste l'ajout d'une fine couche vide.

## Structure interne d'une image (manifest.json)

Une image exportée suit le format **OCI Image Layout** et peut être inspectée directement, sans outil supplémentaire :

```bash
docker save nginx:alpine -o nginx-alpine.tar
mkdir nginx-image && tar -xf nginx-alpine.tar -C nginx-image
```

L'archive extraite contient :

| Élément | Contenu |
|---------|---------|
| `manifest.json` | Décrit la composition de l'image : configuration, liste des couches, tags |
| `blobs/sha256/<hash>` | Chaque couche et chaque fichier de configuration, nommé par son propre hash SHA256 |
| `repositories` | Nom et tag de l'image (`nginx:alpine`) |

```json
{
  "Config": "blobs/sha256/1ff4bb...",
  "RepoTags": ["nginx:alpine"],
  "Layers": [
    "blobs/sha256/08000c18...",
    "blobs/sha256/c1761f3c..."
  ]
}
```

Le champ `Config` pointe vers le blob JSON qui contient les instructions Dockerfile compilées (CMD, variables d'environnement, volumes, labels...) ; `Layers` liste les couches de fichiers dans l'ordre où elles s'appliquent, de la base vers le sommet.

> [!tip] C'est ce que fait Dive automatiquement
> Explorer manuellement `blobs/sha256/` couche par couche donne exactement l'information que [[Docker 09 — Outils d'analyse & linting]] (Dive) affiche de façon interactive — utile pour comprendre le mécanisme avant de s'appuyer sur l'outil.

## Cas particuliers

> [!warning] La couche du conteneur disparaît avec lui
> Toutes les données écrites dans la couche en lecture-écriture d'un conteneur sont **perdues** quand ce conteneur est supprimé (`docker rm`). Pour des données qui doivent survivre, voir [[Docker 05 — Volumes & persistance]].

> [!tip] Une image = un cache de build
> Le fait que les couches soient réutilisables est la base du système de cache de `docker build` : si une couche n'a pas changé depuis le dernier build, Docker la réutilise au lieu de la reconstruire. Détails dans [[Docker 03 — Dockerfile]].

> [!info] Driver de stockage
> `overlay2` est le driver recommandé sur Linux moderne (noyau ≥ 4.0). D'anciens drivers (`aufs`, `devicemapper`) existent mais sont obsolètes sur les installations actuelles.
