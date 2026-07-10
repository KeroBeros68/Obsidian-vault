#devops #builders #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Outil de build** | Logiciel transformant un Dockerfile (ou équivalent) et un contexte de fichiers en image OCI. |
| **BuildKit** | Builder moderne de Docker, activé par défaut depuis Docker 23.0 ; utilisable aussi en standalone via `buildkitd`. |
| **buildkitd** | Démon BuildKit exécutable indépendamment de Docker. |
| **Buildah** | Outil de build de l'écosystème Red Hat/Podman, rootless et sans daemon, avec un mode script en plus du Dockerfile classique. |
| **`buildah bud`** | Commande Buildah construisant une image à partir d'un Dockerfile classique (*build using dockerfile*). |
| **Kaniko** | Outil Google construisant des images dans un pod Kubernetes standard, sans daemon Docker ni privilèges root. |
| **Cloud Native Buildpacks (CNB)** | Standard (CNCF) de détection automatique de stack applicative et de génération d'image sans Dockerfile. |
| **Builder (Buildpacks)** | Image contenant un ensemble de buildpacks capables de détecter et construire une stack donnée (ex. `paketobuildpacks/builder:base`) — à ne pas confondre avec le sens générique d'« outil de build ». |
| **Pack CLI** | Outil en ligne de commande (`pack build`) pilotant la construction d'une image via Buildpacks. |
| **Bake** | Outil BuildKit définissant des builds complexes de façon déclarative en HCL (`docker-bake.hcl`). |
| **HCL** | *HashiCorp Configuration Language* — langage déclaratif utilisé par Bake pour définir targets, groups et variables de build. |
| **Cache registry** | Stratégie de cache de build partagée via un registry (`--cache-from`/`--cache-to type=registry`), au lieu du cache local d'une seule machine. |
| **SBOM (Software Bill of Materials)** | Liste des composants logiciels d'une image, générée automatiquement par les Buildpacks (absente par défaut avec BuildKit/Buildah/Kaniko). |
| **Rootless (build)** | Capacité à construire une image sans privilèges root — natif chez Buildah et Kaniko, possible mais non par défaut avec BuildKit. |
