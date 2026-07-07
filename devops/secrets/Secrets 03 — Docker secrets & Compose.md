#devops #secrets #docker

## Deux implémentations, un seul mot "secret"

Docker propose deux mécanismes distincts qui partagent la même syntaxe `secrets:`, mais pas les mêmes garanties.

| | Docker Compose (hors Swarm) | Docker Swarm |
|---|-------------------------------|----------------|
| Stockage | Fichier local monté tel quel | Chiffré au repos dans le magasin Raft du cluster |
| Distribution | Aucune — le fichier doit déjà exister sur la machine | Distribué automatiquement aux nœuds qui en ont besoin |
| Chiffrement en transit | N/A (montage local) | ✅ TLS entre les nœuds du cluster |
| Cas d'usage typique | Développement, mono-hôte | Cluster multi-nœuds en production |

```yaml
services:
  db:
    image: mariadb:11
    secrets:
      - db_root_password

secrets:
  db_root_password:
    file: ./secrets/db_root_password.txt   # Compose : simple bind-mount du fichier
```

```bash
# Swarm : le secret est stocké chiffré par le cluster, pas dans un fichier local
echo "hunter2" | docker secret create db_root_password -
docker service create --secret db_root_password mariadb:11
```

## Ce que Compose (hors Swarm) ne fait pas

En Compose classique, `secrets:` n'ajoute **aucun chiffrement** : c'est une manière standardisée de monter un fichier en lecture seule sous `/run/secrets/<nom>`, rien de plus. Le fichier source (`./secrets/db_root_password.txt`) reste un fichier en clair sur le disque de la machine hôte, avec les mêmes risques que n'importe quel fichier sensible.

## Secrets au moment du build, via Compose

Les `secrets:` vus jusqu'ici s'appliquent au **runtime** (conteneur déjà démarré). Compose peut aussi transmettre un secret au **build**, relayé vers `docker build --secret` (voir [[Secrets 04 — .dockerignore & hygiène de build]]) :

```yaml
services:
  app:
    build:
      context: .
      secrets:
        - npm_token

secrets:
  npm_token:
    environment: NPM_TOKEN   # lit la valeur depuis la variable d'environnement du shell qui lance `docker compose build`
```

Le Dockerfile associé utilise `RUN --mount=type=secret,id=npm_token` exactement comme en ligne de commande — Compose se contente d'automatiser le passage du `--secret` au build.

## Cas particuliers

> [!warning] Le fichier source doit être protégé lui-même
> `secrets:` en Compose déplace le problème (du process vers un fichier) sans le résoudre par chiffrement. Le fichier source doit être exclu de Git (`.gitignore`), avoir des permissions restrictives (`chmod 600`), et idéalement être généré à partir d'un gestionnaire externe plutôt qu'écrit à la main. Voir [[Secrets 06 — Gestionnaires de secrets externes]].

> [!tip] Compose reste suffisant pour du mono-hôte
> Pour un déploiement sur une seule machine (petit projet, environnement de dev/staging), la simplicité de Compose est un bon compromis tant que le fichier source est correctement protégé. Le passage à Swarm ou Kubernetes se justifie surtout par le besoin de distribuer les secrets entre plusieurs nœuds.

> [!info] Kubernetes suit une logique proche
> Un `Secret` Kubernetes est, par défaut, encodé en base64 (pas chiffré) et stocké dans `etcd` — un chiffrement au repos réel nécessite une configuration explicite (`EncryptionConfiguration`) ou un fournisseur externe (Vault, KMS cloud). Voir [[Manques]] pour Kubernetes (non couvert).
