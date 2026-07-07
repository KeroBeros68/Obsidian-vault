#devops #conteneurs #oci

## Qu'est-ce que l'OCI ?

L'**Open Container Initiative (OCI)** est l'organisation qui définit les standards ouverts sur lesquels reposent tous les moteurs de conteneurs modernes (voir [[Conteneurs — Moteurs]]).

| Spécification | Ce qu'elle définit |
|----------------|----------------------|
| **OCI Image** | Format des images : layers, config, manifest |
| **OCI Runtime** | Comment exécuter un conteneur (implémenté par `runc`) |
| **OCI Distribution** | API des registries : push, pull, list |

> [!info] OCI 1.1 (mars 2024)
> Cette version introduit le champ `subject` et la **Referrers API**, qui permettent d'associer formellement des signatures, SBOM et attestations à une image (voir plus bas) — support encore variable selon les registries.

Grâce à ce standard, une image construite avec Docker s'exécute avec Podman, containerd ou CRI-O, et un registry comme Harbor ou GHCR parle le même langage que Docker Hub.

## Anatomie d'une image : tout est adressé par son contenu

Une image OCI est composée de trois types d'objets, chacun identifié par son **digest** (hash SHA256 de son propre contenu) :

| Objet | Contenu |
|-------|---------|
| **Manifest** | Référence vers la config et la liste des layers |
| **Config** | Métadonnées (variables d'env, commande, user, ports) |
| **Layers** | Fichiers de chaque couche (`tar.gz`) |

| Approche | Exemple | Caractéristique |
|----------|---------|-------------------|
| Location-addressable (classique) | `https://example.com/image.tar` | Même URL, contenu qui peut changer |
| Content-addressable (OCI) | `sha256:abc123...` | Même hash = même contenu garanti |

Ce mode d'adressage par le contenu (et non par un emplacement) permet la déduplication des layers identiques entre images, une vérification d'intégrité par recalcul du hash, et une immutabilité garantie : un digest référence toujours le même contenu.

## Tag vs digest : la différence fondamentale

Un **tag** (`nginx:1.25`) est un alias lisible qui pointe vers un manifest — comme un signet, il peut être déplacé vers un contenu différent (après un rebuild, par exemple). Un **digest** (`nginx@sha256:6926dd80...`) est le hash du manifest lui-même : une empreinte qui identifie une version exacte, de façon permanente.

| Contexte | Utiliser | Exemple |
|----------|----------|---------|
| Développement local | Tag | `nginx:1.25` |
| CI/CD (publication) | Tag versionné + capture du digest | `mon-app:1.2.3` → `sha256:abc...` |
| Production (déploiement) | Digest | `mon-app@sha256:abc...` |
| Audit / conformité | Digest | traçabilité exacte |

> [!warning] Un tag versionné n'est pas forcément immuable
> Même un tag qui ressemble à une version figée (`v1.2.3`) peut être republié avec un contenu différent, sauf si le registry applique une policy d'immutabilité. La seule garantie d'immutabilité vient du digest, pas de la forme du tag.

## Le manifest, cœur de l'image

Le manifest est un fichier JSON qui liste le digest de la config et le digest de chaque layer :

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:aaa...",
    "size": 1234
  },
  "layers": [
    { "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip", "digest": "sha256:bbb...", "size": 5678 },
    { "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip", "digest": "sha256:ccc...", "size": 9012 }
  ]
}
```

Les registries acceptent encore deux `mediaType` de manifest : le format Docker v2 historique (`application/vnd.docker.distribution.manifest.v2+json`) et le format OCI actuel (`application/vnd.oci.image.manifest.v1+json`) — la plupart des outils gèrent les deux de façon transparente.

## Multi-arch : une référence, plusieurs architectures

Un **manifest list** (ou *OCI image index*) est un manifest spécial qui référence un manifest distinct par architecture, plutôt que des layers directement. Quand `docker pull nginx:1.25` s'exécute :

```
1. Télécharge le manifest list (l'index)
2. Identifie l'architecture locale (ex. linux/amd64)
3. Télécharge le manifest spécifique à cette architecture
4. Télécharge les layers correspondants
```

> [!warning] Le digest de l'index diffère du digest de chaque architecture
> Le digest d'un tag multi-arch (`sha256:multi123...`) n'est pas le même que le digest du manifest `amd64` ou `arm64` qu'il référence. Pinner le digest de l'index conserve le choix automatique d'architecture ; pinner le digest d'une architecture spécifique la fige définitivement. Une confusion entre les deux en CI/CD est une source classique d'échecs de déploiement multi-plateforme.

## Artefacts OCI : au-delà des images

Un registry OCI peut aussi stocker d'autres types d'artefacts que des images de conteneurs :

| Artefact | Cas d'usage | Outil |
|----------|-------------|-------|
| Helm charts | Packages Kubernetes | `helm push` |
| SBOM | Liste des composants logiciels | Syft, ORAS |
| Signatures | Attestation d'origine | Cosign, Notation |
| Policies | Règles OPA/Rego | Conftest |

Depuis OCI 1.1, le champ `subject` et la **Referrers API** (`GET /v2/<name>/referrers/<digest>`) permettent d'associer formellement ces artefacts à l'image qu'ils concernent, et de les retrouver tous à partir de son digest.

> [!info] Support variable selon les registries
> La Referrers API est implémentée de façon inégale (Docker Hub, Harbor ≥ 2.9, support progressif sur GHCR/ECR/ACR). À tester explicitement avant de s'appuyer dessus en production.

## Outils CLI pour manipuler des images OCI

| Outil | Force |
|-------|-------|
| `crane` | Léger, rapide, interroge le registry sans pull complet |
| `skopeo` | Copie d'images entre registries, sans daemon |
| `regctl` | Nettoyage de tags, garbage collection |
| `oras` | Pousser/puller des artefacts génériques, Referrers API |

```bash
# Obtenir le digest d'un tag sans puller l'image (registry direct)
crane digest nginx:1.25

# Copier une image entre registries sans Docker daemon
skopeo copy docker://nginx:1.25 docker://mon-registry/nginx:1.25
```

## Cas particuliers

> [!tip] Séparer publication et déploiement en CI/CD
> Un pattern courant : builder et pousser avec un tag lisible (`docker buildx build --tag ... --push`), puis capturer le digest correspondant (`crane digest ...`) et déployer avec ce digest (`kubectl set image ... app=image@sha256:...`). Le tag reste lisible pour les humains, le digest garantit la reproductibilité du déploiement.

> [!warning] Émulation multi-arch sur Apple Silicon
> Builder une image `linux/amd64` sur un Mac ARM64 (ou l'inverse) passe par l'émulation QEMU, 5 à 10 fois plus lente qu'un build natif. Pour la CI, préférer des runners natifs par architecture.

## Pour aller plus loin

Cette fiche prolonge [[Docker 04 — Registry & distribution]] sur la structure interne du format d'image. La sécurisation de la chaîne d'approvisionnement des images (scan de vulnérabilités, SBOM, signatures) n'est pas couverte en tant que module dédié — voir [[Manques]].

Sources : [Comprendre OCI — Stéphane Robert](https://blog.stephane-robert.info/docs/conteneurisation/oci/)
