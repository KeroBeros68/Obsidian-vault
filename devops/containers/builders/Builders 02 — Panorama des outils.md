#devops #builders #comparatif

## Cinq outils, cinq contextes

| Outil | En une phrase |
|-------|-------------------|
| **BuildKit** | Le standard — intégré à Docker, activé par défaut depuis Docker 23.0 |
| **Buildah** | Rootless et sans daemon, écosystème Red Hat/Podman |
| **Kaniko** | Construire des images dans Kubernetes, sans daemon ni privilèges |
| **Buildpacks** | Zéro Dockerfile — détection automatique de la stack applicative |
| **Bake** | Builds déclaratifs en HCL, au-dessus de BuildKit |

## Tableau comparatif détaillé

| Critère | BuildKit | Buildah | Kaniko | Buildpacks | Bake |
|---------|----------|---------|--------|------------|------|
| Daemon requis | Oui (`dockerd`/`buildkitd`) | ❌ Non | ❌ Non | Oui (`dockerd`) | Oui (BuildKit) |
| Rootless | ⚠️ Possible | ✅ Par défaut | ✅ Oui | ⚠️ Selon setup | ⚠️ Selon setup |
| Format d'entrée | Dockerfile | Dockerfile + mode script | Dockerfile | Auto-détection | HCL (+ Dockerfile) |
| Cache avancé | ✅ Excellent | ⚠️ Basique | ✅ Registry | ✅ Rebase des layers | ✅ Excellent |
| Multi-arch | ✅ QEMU intégré | ⚠️ Manuel | ⚠️ Builds séparés | ✅ Natif | ✅ QEMU intégré |
| Secrets sécurisés | ✅ `--mount=type=secret` | ✅ `--secret` | ✅ Via secrets K8s | ✅ Buildpack API | ✅ Idem BuildKit |
| Cache registry | ✅ `--cache-from/to` | ⚠️ Limité | ✅ Natif | ⚠️ Non | ✅ Idem BuildKit |
| Environnement cible | Local + CI/CD | Local + CI/CD | Kubernetes | Local + PaaS | Local + CI/CD |
| SBOM automatique | ❌ Non | ❌ Non | ❌ Non | ✅ Oui | ❌ Non |
| Courbe d'apprentissage | ⚠️ Moyenne | ⚠️ Moyenne | ⚠️ Moyenne | ✅ Facile | ⚠️ Élevée |

## Recommandation par contexte

| Situation | Outil recommandé | Pourquoi |
|-----------|----------------------|----------|
| Développement local (Docker) | BuildKit | Intégré, cache performant, multi-arch |
| Développement local (Podman) | Buildah | Rootless, sans daemon, natif à Podman |
| CI/CD sur Kubernetes | Kaniko | Sans daemon, unprivileged, cache registry |
| CI/CD multi-langage | Buildpacks | Détection auto de la stack, pas de Dockerfile |
| Builds complexes (mono-repo) | Bake | Déclaratif HCL, targets, groups, variables |
| Sécurité maximale (rootless strict) | Buildah | Rootless natif, sans daemon |
| Multi-arch régulier | BuildKit | QEMU intégré, builds parallèles |

> [!tip] Le choix par défaut
> Si Docker est déjà utilisé, BuildKit est déjà installé et activé — c'est le meilleur compromis performance/fonctionnalités pour la plupart des cas. Les autres outils s'imposent pour une contrainte précise (Kubernetes sans daemon, rootless strict, zéro Dockerfile).

## Détail rapide de chaque outil

- **BuildKit** — cache intelligent, builds parallèles, multi-arch natif, cache registry, secrets via `--mount=type=secret`. Limite : nécessite un daemon (Docker ou `buildkitd` standalone), pas rootless par défaut.
- **Buildah** — rootless natif, sans daemon, deux modes (Dockerfile classique ou scripting `buildah from`/`run`/`commit`), `podman build` l'utilise en interne. Limite : écosystème et cache moins matures que BuildKit.
- **Kaniko** — tourne dans un pod Kubernetes standard, unprivileged, cache via registry, image reproductible. Limite : pas de parallélisme, pas de multi-arch natif, pensé exclusivement pour un environnement conteneurisé (pas pour un poste local).
- **Buildpacks** — détection automatique de la stack (Node, Python, Java...), bonnes pratiques incluses, SBOM généré, rebasing de l'OS sans rebuild complet. Limite : moins de contrôle, images parfois plus lourdes.
- **Bake** — définit des builds complexes en HCL (`docker-bake.hcl`), targets multiples avec héritage, peut aussi lire un `docker-compose.yml`. Limite : nécessite BuildKit, moins connu qu'un Dockerfile classique.

## Cas particuliers

> [!info] Compatibilité OCI garantie entre tous les outils
> Une image construite avec Buildah s'exécute avec Docker, Podman ou containerd sans modification — voir [[Builders 01 — Qu'est-ce qu'un outil de build]] et [[Conteneurs — OCI]].

> [!warning] Kaniko n'est pas fait pour un poste de développement local
> Techniquement exécutable via Docker en local, mais plus lent et moins ergonomique que BuildKit ou Buildah pour cet usage — Kaniko est conçu pour tourner dans un conteneur (Kubernetes, GitLab CI), pas sur un poste de dev.

> [!tip] Le cache est ce qui change tout en CI/CD
> Sans `--cache-from`/`--cache-to type=registry` (BuildKit) ou `--cache-repo` (Kaniko), chaque build en CI repart de zéro — le gain de temps d'un cache registry partagé entre runs est souvent le facteur le plus impactant, avant même le choix de l'outil.
