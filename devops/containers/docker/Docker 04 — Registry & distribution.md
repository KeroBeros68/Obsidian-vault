#devops #docker #registry #fondamentaux

## Le registry, entrepôt des images

Une image construite localement ne vit que sur la machine qui l'a buildée. Un **registry** est un serveur qui stocke et distribue des images, pour les rendre accessibles à d'autres machines (CI, serveurs de production, collègues).

| Registry | Particularité |
|----------|---------------|
| **Docker Hub** (`docker.io`) | Registry par défaut — implicite si aucun n'est précisé |
| **GitHub Container Registry** (`ghcr.io`) | Lié à un compte/organisation GitHub |
| **Registries cloud** (ECR, ACR, Artifact Registry...) | Intégrés aux écosystèmes AWS, Azure, GCP |
| **Registry privé auto-hébergé** | Contrôle total, déploiement interne |

## Anatomie d'un nom d'image

```
registry.example.com/team/service:1.0.0
└────────┬──────────┘ └─┬─┘ └──┬──┘ └─┬─┘
      registry        namespace  image  tag
```

- Si le registry n'est pas précisé, Docker Hub est implicite : `nginx` équivaut à `docker.io/library/nginx`.
- Pour tout autre registry, le préfixe complet est obligatoire — sans lui, Docker cherche sur Docker Hub par défaut, pas sur le registry privé.

```bash
# Ces deux commandes sont strictement équivalentes
docker pull nginx
docker pull docker.io/library/nginx:latest
```

## Le cycle build → tag → push → pull

```bash
# 1. Construire l'image localement
docker build -t myapp:1.0.0 .

# 2. Associer un nom complet pointant vers le registry cible
docker tag myapp:1.0.0 registry.example.com/team/myapp:1.0.0

# 3. S'authentifier auprès du registry
docker login registry.example.com

# 4. Envoyer l'image vers le registry
docker push registry.example.com/team/myapp:1.0.0

# 5. Depuis une autre machine : récupérer l'image
docker pull registry.example.com/team/myapp:1.0.0
```

## Tags : versions mutables vs immuables

- Un **tag** est une étiquette lisible (`v1.2.3`, `latest`) qui pointe vers une version précise d'une image.
- Un tag est **mutable par défaut** : repousser une nouvelle image avec le même tag déplace l'étiquette vers cette nouvelle version — l'historique n'est pas conservé sous ce nom.
- Le **digest** (`@sha256:...`) identifie une image de façon immuable et unique : il ne change jamais, contrairement à un tag.

```bash
# Tag mutable — peut changer de contenu dans le temps
docker pull myapp:latest

# Digest immuable — pointe toujours exactement vers la même image
docker pull myapp@sha256:abc123def456...
```

## Cas particuliers

> [!warning] latest n'est pas une version
> `latest` n'indique aucune garantie de stabilité ni de contenu précis — c'est juste le tag appliqué par défaut si aucun n'est spécifié. En production, toujours utiliser une version explicite (`1.2.3`) ou un SHA de commit.

> [!tip] Plusieurs tags pour une même image
> Une bonne pratique courante consiste à pousser plusieurs tags pour la même image : un tag précis (`1.2.3`) pour la traçabilité, et des tags plus larges (`1.2`, `1`) pour ceux qui veulent automatiquement les correctifs mineurs.

> [!info] Registry implicite
> Si aucun registry n'est précisé dans le nom de l'image, Docker suppose Docker Hub. Toujours qualifier intégralement le nom pour tout autre registry, afin d'éviter toute ambiguïté en script ou en CI.

Pour la structure interne du manifest, le multi-arch et les artefacts OCI (SBOM, signatures), voir [[Conteneurs — OCI]].

## Registry mirrors : cache local pour Docker Hub

Un **registry mirror** agit comme cache : `docker pull` l'interroge en premier, et il ne redescend vers Docker Hub que si l'image n'y est pas déjà. Réduit la bande passante sortante et limite l'exposition aux rate limits de Docker Hub.

```json
{ "registry-mirrors": ["https://mirror.gcr.io"] }
```

Réglage du démon (voir [[Docker 10 — Configuration production & nettoyage]]), pas une option de `docker pull` — vérification : `docker info | grep -A5 "Registry Mirrors"`.

## Registries internes sans HTTPS valide

```json
{ "insecure-registries": ["registry.local:5000", "192.168.1.100:5000"] }
```

> [!warning] `insecure-registries` désactive la vérification TLS
> À ne jamais utiliser pour un registry exposé au-delà d'un réseau strictement contrôlé. Pour un registry interne avec un certificat auto-signé, préférer ajouter son certificat CA plutôt que désactiver la vérification :
> ```bash
> sudo mkdir -p /etc/docker/certs.d/registry.local:5000
> sudo cp ca.crt /etc/docker/certs.d/registry.local:5000/
> ```

## Authentification : credential helpers

`docker login` stocke les identifiants dans `~/.docker/config.json` — un fichier **par utilisateur**, distinct de `daemon.json` (qui s'applique à tout le démon).

```bash
docker login registry.example.com
cat ~/.docker/config.json   # identifiants stockés ici (en clair par défaut, sauf credential helper)
```

Pour éviter un stockage en clair, un **credential helper** délègue l'authentification au gestionnaire d'identifiants du fournisseur cloud :

```json
{
  "credHelpers": {
    "gcr.io": "gcloud",
    "*.dkr.ecr.*.amazonaws.com": "ecr-login"
  }
}
```
