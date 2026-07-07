#devops #conteneurs #moteurs

## Qu'est-ce qu'un moteur de conteneurs ?

Un **moteur de conteneurs** (*container engine*) est le logiciel qui permet de créer, exécuter et gérer des conteneurs. Le terme est cependant utilisé de façon imprécise : Docker, containerd, LXC ne font pas tous la même chose, même s'ils s'appuient tous sur les mêmes fondations Linux (voir [[Conteneurs — Généralités]]).

| Catégorie | Exemples | Rôle |
|-----------|----------|------|
| Engine (moteur complet) | Docker, Podman | Build + run + interface utilisateur |
| Runtime (exécution seule) | containerd, CRI-O | Exécution uniquement, orienté CRI/Kubernetes |
| Build tools | BuildKit, Buildah, Kaniko | Construction d'images uniquement |
| Conteneurs système | LXC, Incus | OS complet dans un conteneur (logique "VM légère") |

## Panorama : les 6 moteurs comparés

| Moteur | Type | Rootless/Unprivileged | Build intégré | Cas d'usage principal |
|--------|------|------------------------|----------------|------------------------|
| **Docker** | Applicatif | Optionnel | ✅ Dockerfile | Développement, apprentissage, CI/CD |
| **Podman** | Applicatif | ✅ Par défaut (Linux) | ✅ Containerfile (via Buildah) | Sécurité, environnements réglementés |
| **containerd** | Runtime | Selon setup | ❌ (externe : BuildKit) | Runtime officiel Kubernetes |
| **CRI-O** | Runtime | Selon setup | ❌ | Kubernetes uniquement (OpenShift) |
| **LXC** | Système | ✅ Unprivileged | ❌ | Migration VM → conteneur, conteneurs multi-services |
| **Incus** | Système + VM | ✅ Par défaut | ❌ | Infra hybride conteneurs/VM, alternative à Proxmox |

> [!tip] Le choix par défaut pour débuter
> Docker reste le standard de facto : documentation la plus abondante, écosystème le plus large. Les images et commandes étant compatibles entre moteurs (standard OCI), une migration vers Podman ou containerd reste possible plus tard sans tout réapprendre.

## Kubernetes et le runtime : pourquoi containerd ou CRI-O ?

Docker et Podman sont des moteurs complets (build + run). Kubernetes, lui, ne dialogue qu'avec un **runtime** conforme à la CRI (*Container Runtime Interface*) — Docker n'implémente pas cette interface nativement.

```
┌─────────────────────────────────────────────┐
│                  Kubernetes                  │
│  ┌─────────┐     ┌─────────┐                 │
│  │ kubelet │────▶│   CRI   │ (interface)      │
│  └─────────┘     └────┬────┘                 │
│                       │                       │
│         ┌─────────────┼─────────────┐        │
│         ▼             ▼             ▼        │
│    containerd       CRI-O      ❌ Docker      │
│    (natif CRI)    (natif CRI)  (via shim)    │
└─────────────────────────────────────────────┘
```

Depuis **Kubernetes 1.24** (mai 2022), l'adaptateur `dockershim` qui permettait à Kubernetes de piloter Docker a été retiré du code source. Kubernetes n'a pas "arrêté Docker" — il a supprimé un composant de traduction devenu à sa charge. Les images construites avec `docker build` restent utilisables sans modification : seul le runtime tournant sur les nœuds du cluster change (containerd ou CRI-O à la place de Docker).

## Rootless vs unprivileged : deux mécanismes, un résultat voisin

| | Rootless (Docker/Podman) | Unprivileged (LXC/Incus) |
|---|---------------------------|----------------------------|
| Mécanisme | Le daemon et les conteneurs tournent sans privilèges root côté hôte | User namespaces : le root du conteneur est mappé sur un UID non privilégié de l'hôte |
| Ce que ça empêche | Un daemon compromis n'a pas les privilèges root de l'hôte | Un attaquant root **dans** le conteneur n'a que les droits d'un UID non privilégié **sur** l'hôte |
| Résultat sécurité | Similaire dans les deux cas : pas d'escalade de privilèges directe vers l'hôte | |

## Cas particuliers

> [!warning] Portabilité OCI ≠ stack runtime identique
> Le standard **OCI** (*Open Container Initiative*) garantit qu'une image construite avec Docker s'exécute avec Podman, containerd ou CRI-O sans modification. Il ne garantit pas que le reste de la stack soit identique : drivers réseau (CNI vs réseau Docker), gestion des volumes, contextes de sécurité (SELinux, AppArmor) peuvent différer d'un moteur à l'autre. Détail du format d'image dans [[Conteneurs — OCI]].

> [!info] `nerdctl` n'est pas un remplacement de Docker
> `containerd` expose nativement une CLI minimale (`ctr`), peu conviviale. `nerdctl` est une CLI compatible Docker qui pilote `containerd` directement (`nerdctl run`, `nerdctl build`, `nerdctl compose`) — un confort d'usage, pas un moteur différent.

> [!tip] Compatibilité Podman ↔ Docker
> `podman` accepte en grande partie les mêmes commandes que `docker` (`podman run`, `podman build`, `podman ps`), au point qu'un simple alias `docker=podman` suffit souvent à migrer un poste de développement. Les Dockerfiles fonctionnent tels quels (Podman les appelle *Containerfile*).

## Pour aller plus loin

Ce comparatif prépare le choix d'un moteur ; la mise en pratique du moteur retenu par ce vault continue avec [[Docker — Index des fiches]]. Podman, containerd et Kubernetes ne sont pas couverts en modules dédiés pour l'instant — voir [[Manques]].

Sources : [Moteurs de conteneurs — Stéphane Robert](https://blog.stephane-robert.info/docs/conteneurs/moteurs-conteneurs/)
