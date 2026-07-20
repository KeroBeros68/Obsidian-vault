#devops #conteneurs #fondamentaux

## Conteneurisation vs virtualisation

Deux façons d'isoler des environnements d'exécution, avec des philosophies différentes :

- **Virtualisation** : plusieurs systèmes d'exploitation complets et indépendants tournent sur une même machine physique, via un **hyperviseur**. Chaque VM embarque son propre noyau — isolation forte, mais lourde et lente à démarrer.
- **Conteneurisation** : isole uniquement l'application et ce dont elle a besoin (code, dépendances, configuration). Les conteneurs **partagent le noyau de l'hôte**, ce qui les rend beaucoup plus légers et rapides, au prix d'une isolation moins étanche qu'une VM.

| Critère | Virtualisation (VMs) | Conteneurisation |
|---------|----------------------|-------------------|
| Isolation | OS complet isolé | Processus isolé, noyau partagé |
| Taille | Plusieurs Go | Quelques Mo à centaines de Mo |
| Démarrage | Minutes | Secondes |
| Ressources | Élevées (RAM, CPU dédiés) | Légères (partagées) |
| Portabilité | Dépend de l'hyperviseur | Très portable (image OCI standard) |
| Cas d'usage | Multi-OS, isolation forte | Microservices, CI/CD, cloud-native |

## Les trois piliers Linux de l'isolation

La conteneurisation ne virtualise rien : elle s'appuie sur trois fonctionnalités du noyau Linux qui, combinées, donnent l'illusion d'une machine isolée alors qu'il s'agit d'un simple processus.

- **Namespaces** : isolent la *vue* qu'un processus a des ressources du système — un namespace PID donne à un conteneur ses propres identifiants de processus, un namespace réseau lui donne sa propre pile réseau (interfaces, règles de pare-feu), indépendants de l'hôte et des autres conteneurs.
- **Cgroups** (*control groups*) : limitent et mesurent la *consommation* de ressources (CPU, mémoire, I/O) d'un groupe de processus, pour qu'un conteneur ne monopolise pas les ressources de l'hôte.
- **Capabilities** : fragmentent les privilèges du root traditionnel en unités indépendantes (ex. modifier l'horloge système sans pour autant pouvoir tout faire), pour donner à un conteneur seulement les droits dont il a réellement besoin.

Mise en pratique concrète (commandes d'inspection, seccomp comme quatrième pilier, cgroups v1/v2) dans [[Docker 11 — Sous le capot (namespaces, cgroups, seccomp)]].

## Illustration

```
Virtualisation                          Conteneurisation
┌─────────────────────────┐            ┌─────────────────────────┐
│         VM 1  │  VM 2   │            │ Conteneur 1 │ Conteneur 2│
│  ┌─────────┐ │┌───────┐│            │  (namespaces + cgroups   │
│  │ Noyau A │ ││Noyau B││            │   + capabilities)        │
│  └─────────┘ │└───────┘│            ├─────────────────────────┤
├─────────────────────────┤            │      Noyau de l'hôte     │
│        Hyperviseur       │            ├─────────────────────────┤
├─────────────────────────┤            │      Machine physique    │
│      Machine physique    │            └─────────────────────────┘
└─────────────────────────┘
```

Namespaces, cgroups et capabilities agissent ensemble : les namespaces isolent ce qu'un processus *voit*, les cgroups limitent ce qu'il *consomme*, les capabilities restreignent ce qu'il a le *droit de faire*.

## Panorama de l'écosystème

La conteneurisation se découpe en couches indépendantes, chacune avec plusieurs outils concurrents :

| Couche | Rôle | Exemples |
|--------|------|----------|
| Moteur de conteneurs | Construire et exécuter les conteneurs | Docker, Podman (rootless, sans daemon), LXC/Incus (conteneurs "système") |
| Registry | Stocker et distribuer les images | Docker Hub, Harbor, GitLab Container Registry |
| Orchestrateur | Piloter des conteneurs à grande échelle (résilience, scalabilité, réseau) | Docker Compose (dev local, pas un orchestrateur), Docker Swarm, Kubernetes, Nomad, K3s |

## Cas particuliers

> [!warning] Noyau partagé = isolation moins étanche qu'une VM
> Une faille noyau ou une mauvaise configuration de capabilities peut permettre à un processus conteneurisé d'affecter l'hôte ou d'autres conteneurs, puisqu'ils partagent le même noyau — une VM, avec son propre noyau, n'expose pas cette surface. C'est pourquoi la réduction des privilèges (voir [[Docker 08 — Sécurité des conteneurs]]) n'est pas optionnelle en production.

> [!tip] Docker Compose n'est pas un orchestrateur
> Compose définit et lance plusieurs conteneurs en local, mais ne gère ni la scalabilité automatique, ni le redémarrage en cas de panne, ni la distribution sur plusieurs machines — voir [[Docker 07 — Docker Compose]]. Pour la production à grande échelle, c'est Kubernetes, Swarm ou Nomad qui prennent le relais (non couverts, voir [[Manques]]).

> [!info] Portabilité via le format OCI
> Le format d'image standardisé par l'**Open Container Initiative (OCI)** est ce qui rend une image construite avec un outil (Docker, Buildah, Kaniko) exécutable par n'importe quel moteur compatible OCI (Docker, Podman, containerd) — la portabilité ne dépend pas de l'outil qui a construit l'image. Détails dans [[Conteneurs — OCI]].

## Pour aller plus loin

Ce panorama pose les concepts communs à tout le domaine ; le choix d'un moteur en particulier est détaillé dans [[Conteneurs — Moteurs]], le format d'image standardisé dans [[Conteneurs — OCI]], le vocabulaire complet dans [[Conteneurs — Glossaire]], et la mise en pratique concrète commence avec [[Docker — Index des fiches]].

Sources : [Maîtriser la conteneurisation — Stéphane Robert](https://blog.stephane-robert.info/docs/conteneurisation/)
