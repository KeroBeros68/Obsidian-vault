#devops #conteneurs #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Virtualisation** | Isolation par hyperviseur : plusieurs systèmes d'exploitation complets, chacun avec son propre noyau, tournent sur une même machine physique. |
| **Hyperviseur** | Logiciel qui crée et gère des machines virtuelles, en leur donnant chacune l'illusion de matériel dédié. |
| **Conteneurisation** | Isolation au niveau processus : les conteneurs partagent le noyau de l'hôte, plus légers et rapides qu'une VM, au prix d'une isolation moins étanche. |
| **Namespace (Linux)** | Mécanisme noyau isolant la *vue* qu'un processus a d'une ressource système (PID, réseau, montages...) — un des trois piliers de l'isolation des conteneurs. |
| **Cgroup (control group)** | Mécanisme noyau limitant et mesurant la *consommation* de ressources (CPU, mémoire, I/O) d'un groupe de processus. |
| **Capability (Linux)** | Fragment de privilège root traditionnel, attribuable ou retirable indépendamment des autres — permet de donner à un conteneur seulement les droits dont il a besoin. |
| **Moteur de conteneurs (engine)** | Logiciel complet capable de construire et d'exécuter des conteneurs (ex. Docker, Podman) — à distinguer d'un simple runtime. |
| **Runtime** | Composant assurant uniquement l'exécution des conteneurs, sans fonction de build (ex. containerd, CRI-O) — utilisé notamment par Kubernetes. |
| **Conteneurs système** | Conteneurs qui embarquent un OS complet plutôt qu'un seul processus applicatif (ex. LXC, Incus) — logique de "VM légère". |
| **Registry** | Serveur qui stocke et distribue des images de conteneurs (Docker Hub, Harbor, GitLab Container Registry...). |
| **Orchestrateur** | Système qui pilote des conteneurs à grande échelle : résilience, scalabilité, réseau (Kubernetes, Swarm, Nomad) — Docker Compose n'en est pas un. |
| **CRI (Container Runtime Interface)** | Interface standard que Kubernetes utilise pour dialoguer avec un runtime de conteneurs — Docker ne l'implémente pas nativement. |
| **`dockershim`** | Adaptateur qui permettait à Kubernetes de piloter Docker malgré l'absence de support CRI natif ; retiré depuis Kubernetes 1.24. |
| **Rootless** | Le démon et les conteneurs tournent sans privilèges root côté hôte (Docker, Podman). |
| **Unprivileged** | Le root à l'intérieur d'un conteneur système est mappé, via les user namespaces, sur un UID non privilégié de l'hôte (LXC, Incus) — résultat de sécurité proche du rootless, mécanisme différent. |
| **`nerdctl`** | CLI compatible Docker pilotant containerd directement — un confort d'usage, pas un moteur différent. |
| **OCI (Open Container Initiative)** | Organisation qui standardise le format d'image, l'exécution et l'API de distribution des conteneurs, garantissant la portabilité entre moteurs. |
| **Content-addressable** | Mode d'adressage où un objet est identifié par le hash de son propre contenu (le digest), garantissant qu'un même identifiant pointe toujours vers le même contenu. |
| **Digest** | Hash SHA256 du manifest d'une image, l'identifiant de façon immuable et unique — contrairement à un tag. |
| **Tag** | Alias lisible pointant vers un manifest, mutable par défaut — peut être redéplacé vers un contenu différent après un rebuild. |
| **Manifest** | Fichier JSON listant le digest de la configuration et de chaque couche d'une image. |
| **Manifest list (OCI image index)** | Manifest spécial référençant un manifest distinct par architecture, permettant à une seule référence de servir plusieurs plateformes (multi-arch). |
| **Referrers API** | API introduite par OCI 1.1 permettant d'associer et de retrouver des artefacts (signatures, SBOM, attestations) liés à une image via son digest. |
| **SBOM (Software Bill of Materials)** | Liste des composants logiciels d'une image, stockable comme artefact OCI. |
