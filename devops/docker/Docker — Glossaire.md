#devops #docker #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Image** | Modèle figé et en lecture seule à partir duquel des conteneurs sont créés. Composée de couches empilées. |
| **Conteneur** | Instance en cours d'exécution d'une image, avec une couche en lecture-écriture propre, supprimée avec lui. |
| **Couche (layer)** | Ensemble de changements filesystem créé par une instruction Dockerfile. Immuable et potentiellement partagée entre images. |
| **Union filesystem** | Technologie qui empile plusieurs couches en une vue unifiée (driver `overlay2` sous Linux moderne). |
| **Copy-on-write** | Stratégie où un fichier d'une couche en lecture seule n'est copié vers la couche du conteneur qu'au moment de sa modification. |
| **Dockerfile** | Fichier texte décrivant la suite d'instructions construisant une image. |
| **Cache de build** | Réutilisation d'une couche déjà construite si l'instruction et ses entrées n'ont pas changé depuis le dernier build. |
| **Multi-stage build** | Dockerfile utilisant plusieurs `FROM`, où seule une partie du résultat d'un stage est copiée vers le stage suivant (`COPY --from=`). |
| **BuildKit** | Moteur de build moderne de Docker, activé par défaut, qui permet des fonctionnalités avancées comme les caches montés (`--mount=type=cache`). |
| **Volume (nommé)** | Espace de stockage géré entièrement par Docker, recommandé pour les données persistantes en production. |
| **Bind mount** | Montage d'un chemin précis de l'hôte dans le conteneur ; pratique en développement, déconseillé en production. |
| **tmpfs** | Montage stocké uniquement en RAM, jamais écrit sur disque ; perdu à l'arrêt du conteneur. |
| **Bridge (réseau)** | Driver réseau par défaut sur un seul hôte. Un bridge personnalisé (créé via `docker network create`) offre la résolution DNS par nom. |
| **Driver host** | Driver réseau qui supprime l'isolation : le conteneur partage directement la pile réseau de l'hôte. |
| **Driver overlay** | Driver réseau reliant plusieurs hôtes Docker entre eux (utilisé par Swarm). |
| **EXPOSE** | Instruction Dockerfile purement documentaire — ne publie aucun port. |
| **Publication de port (`-p` / `ports:`)** | Action réelle qui rend un port du conteneur accessible depuis l'hôte. |
| **Docker Compose** | Outil de description et d'orchestration d'applications multi-conteneurs via un fichier YAML (`compose.yml`). |
| **Service (Compose)** | Définition d'un conteneur dans un fichier Compose (image, build, volumes, réseaux, dépendances). |
