#devops #builders #fondamentaux

## Un outil de build transforme du texte en image OCI

Un **outil de build** transforme un Dockerfile (ou équivalent) et un **contexte** (les fichiers sources) en une **image OCI** prête à être exécutée par un moteur de conteneurs (voir [[Conteneurs — OCI]]).

## Build vs exécution : deux familles d'outils distinctes

| Catégorie | Outils | Produit |
|-----------|--------|---------|
| Construire des images | BuildKit, Buildah, Kaniko | Une image OCI (archive `tar`, ou push vers un registry) |
| Exécuter des conteneurs | Docker, Podman, containerd | Un conteneur en cours d'exécution |

Certains outils combinent les deux rôles : **Docker** assemble BuildKit (build) et containerd (exécution) ; **Podman** assemble Buildah (build) et crun/runc (exécution). Voir [[Conteneurs — Moteurs]] pour la distinction moteur/runtime/build tools au niveau de l'écosystème entier.

## Ce que fait concrètement un outil de build

| Fonction | Description |
|----------|--------------|
| Parser le Dockerfile | Analyser les instructions (`FROM`, `RUN`, `COPY`...) |
| Gérer les layers | Créer une couche par instruction, réutiliser celles déjà construites |
| Exécuter les commandes | Lancer chaque `RUN` dans un environnement temporaire isolé |
| Gérer le cache | Réutiliser les couches identiques d'un build à l'autre |
| Produire l'image finale | Générer le manifest OCI et les couches compressées |
| Pousser vers un registry | Upload de l'image vers Docker Hub, GHCR, un registry privé... |

## Deux philosophies : Dockerfile vs Buildpacks

| | Avec Dockerfile | Sans Dockerfile (Buildpacks) |
|---|--------------------|-----------------------------------|
| Principe | Vous écrivez chaque instruction de build | L'outil détecte automatiquement la stack (`package.json`, `requirements.txt`...) et construit l'image |
| Avantages | Contrôle total, reproductible, versionnable | Zéro configuration, bonnes pratiques incluses, SBOM généré automatiquement |
| Inconvénients | Répétitif, nécessite une expertise, maintenance manuelle | Moins de contrôle, images parfois plus lourdes |

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "index.js"]
```

```bash
# Équivalent Buildpacks : aucune instruction à écrire, la stack est détectée
pack build mon-app --builder paketobuildpacks/builder:base
```

## Cas particuliers

> [!info] Compatibilité OCI garantie, quel que soit l'outil
> Une image construite avec Buildah s'exécute avec Docker, Podman ou containerd sans modification — le format de sortie est standardisé (voir [[Conteneurs — OCI]]), seule la façon de la construire diffère d'un outil à l'autre.

> [!tip] Ne pas confondre les deux sens du mot "builder"
> Un **outil de build** (BuildKit, Buildah, Kaniko) construit des images. Un **builder Buildpacks** (ex. `paketobuildpacks/builder:base`) est une image particulière qui embarque un ensemble de buildpacks et sait détecter/construire une stack donnée — un cas d'usage précis du premier terme, pas un synonyme.

## Pour aller plus loin

Le panorama et le comparatif détaillé des 5 outils (BuildKit, Buildah, Kaniko, Buildpacks, Bake) sont dans [[Builders 02 — Panorama des outils]].
