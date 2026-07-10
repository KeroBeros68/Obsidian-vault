#devops #docker #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Image** | Modèle figé et en lecture seule à partir duquel des conteneurs sont créés. Composée de couches empilées. |
| **Conteneur** | Instance en cours d'exécution d'une image, avec une couche en lecture-écriture propre, supprimée avec lui. |
| **docker exec** | Commande exécutant une instruction dans un conteneur déjà démarré (ne fonctionne pas sur un conteneur arrêté). |
| **docker inspect** | Commande retournant la configuration et l'état complets d'un conteneur (ou image, réseau, volume) au format JSON. |
| **Couche (layer)** | Ensemble de changements filesystem créé par une instruction Dockerfile. Immuable et potentiellement partagée entre images. |
| **Union filesystem** | Technologie qui empile plusieurs couches en une vue unifiée (driver `overlay2` sous Linux moderne). |
| **Copy-on-write** | Stratégie où un fichier d'une couche en lecture seule n'est copié vers la couche du conteneur qu'au moment de sa modification. |
| **Dockerfile** | Fichier texte décrivant la suite d'instructions construisant une image. |
| **Cache de build** | Réutilisation d'une couche déjà construite si l'instruction et ses entrées n'ont pas changé depuis le dernier build. |
| **Multi-stage build** | Dockerfile utilisant plusieurs `FROM`, où seule une partie du résultat d'un stage est copiée vers le stage suivant (`COPY --from=`). |
| **BuildKit** | Moteur de build moderne de Docker, activé par défaut, qui permet des fonctionnalités avancées comme les caches montés (`--mount=type=cache`). |
| **Registry** | Serveur qui stocke et distribue des images Docker (ex. Docker Hub, GitHub Container Registry, registries cloud ou privés). |
| **Repository (image)** | Ensemble d'images partageant le même nom dans un registry, distinguées par leurs tags. |
| **Tag** | Étiquette lisible pointant vers une version précise d'une image. Mutable par défaut : repousser sous le même tag déplace l'étiquette. |
| **Digest** | Identifiant immuable d'une image (`sha256:...`), garantissant qu'on récupère toujours exactement le même contenu, contrairement à un tag. |
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
| **USER (instruction)** | Instruction Dockerfile définissant l'utilisateur sous lequel s'exécutent les instructions suivantes et le processus du conteneur. |
| **Utilisateur non-root** | Utilisateur dédié (UID ≠ 0) sous lequel un conteneur s'exécute, pour limiter l'impact d'une éventuelle compromission. |
| **Capacités Linux (capabilities)** | Privilèges root granulaires (ex. `CAP_NET_BIND_SERVICE`) pouvant être accordés ou retirés individuellement à un conteneur, sans donner un accès root complet. |
| **ARG** | Instruction Dockerfile définissant une variable disponible uniquement au moment du build, absente du conteneur final. |
| **ENV** | Instruction Dockerfile définissant une variable disponible au build et à l'exécution, dans le conteneur final. |
| **ENTRYPOINT** | Instruction Dockerfile définissant l'exécutable fixe du conteneur ; combinée à `CMD`, ce dernier fournit ses arguments par défaut. |
| **HEALTHCHECK / healthcheck** | Commande exécutée périodiquement (Dockerfile ou Compose) pour vérifier qu'un conteneur est réellement opérationnel, au-delà du simple démarrage. |
| **depends_on (condition)** | Option Compose faisant attendre un service jusqu'à ce qu'un autre atteigne un état précis (`service_started`, `service_healthy`). |
| **Secrets (Compose)** | Fichiers montés en lecture seule sous `/run/secrets/` dans les conteneurs qui les référencent, pour transmettre des données sensibles sans les exposer en variable d'environnement. |
| **Ancre YAML (`x-`)** | Bloc réutilisable défini une fois (`x-nom: &ancre`) et référencé ailleurs (`*ancre`), pour éviter la duplication dans un fichier Compose. |
| **Socket Docker (`docker.sock`)** | Interface du démon Docker ; un conteneur y ayant accès peut piloter tous les conteneurs de l'hôte — équivalent fonctionnel à un accès root sur la machine. |
| **musl libc** | Implémentation de la libc utilisée par Alpine, plus légère que glibc mais parfois incompatible avec des binaires précompilés pour glibc. |
| **restart (politique)** | Option Compose/`docker run` définissant si et quand un conteneur redémarre automatiquement (`no`, `always`, `on-failure[:N]`, `unless-stopped`). |
| **IPAM** | *IP Address Management* — plan d'adressage explicite d'un réseau Docker (`subnet`, `gateway`, `ip_range`), à défaut choisi automatiquement. |
| **Réseau internal (Compose)** | Réseau marqué `internal: true` : aucune route vers l'extérieur, les conteneurs qui y sont connectés ne peuvent pas atteindre Internet. |
| **Réseau external (Compose)** | Réseau déjà créé en dehors du fichier Compose (`docker network create` ou un autre projet), réutilisé via `external: true` plutôt que recréé. |
| **Alias réseau** | Nom DNS supplémentaire résolvable pour un service sur un réseau donné, en plus de son nom de service par défaut. |
| **LABEL** | Instruction Dockerfile attachant une métadonnée clé/valeur à l'image, sans effet sur son contenu ni son exécution. |
| **ONBUILD** | Instruction Dockerfile qui différe l'exécution d'une autre instruction au build d'une image **dérivée**, pas à celui de l'image qui la déclare. |
| **VOLUME (instruction Dockerfile)** | Instruction marquant un chemin comme destiné à un volume ; sans montage explicite, Docker y crée un volume anonyme au lancement du conteneur. |
| **STOPSIGNAL** | Instruction Dockerfile définissant le signal envoyé par `docker stop` avant le `SIGKILL` de secours (par défaut `SIGTERM`). |
| **SHELL** | Instruction Dockerfile changeant l'interpréteur utilisé par les formes shell de `RUN`/`CMD`/`ENTRYPOINT` (par défaut `/bin/sh -c`). |
| **Hadolint** | Linter de Dockerfile (règles `DL` + ShellCheck) qui analyse le texte du Dockerfile avant tout build. |
| **Dive** | Outil d'exploration des couches d'une image construite, avec un score d'efficacité et la détection de fichiers dupliqués/gaspillés. |
| **Dockle** | Linter de sécurité d'image construite, basé sur les CIS Docker Benchmarks. |
| **Checkov** | Scanner de sécurité IaC unifié (Dockerfile, Terraform, Kubernetes...) utilisant le même moteur de règles pour plusieurs formats. |
| **CIS Docker Benchmark** | Ensemble de recommandations de sécurité standardisées pour les conteneurs et images Docker, utilisé comme référentiel par des outils comme Dockle. |
| **scratch** | Image de base totalement vide (0 octet) — ne convient qu'à un binaire compilé statiquement, sans libc ni certificats. |
| **distroless (:debug)** | Variante d'une image distroless ajoutant un shell BusyBox absent de l'image de production, réservée au débogage ponctuel. |
| **Chainguard / Wolfi** | Distribution de base (Wolfi) et images associées (Chainguard), reconstruites quotidiennement pour viser 0 CVE connu à la publication. |
| **apko** | Outil de build déclaratif utilisé par Chainguard pour produire des images à partir de la distribution Wolfi. |
| **--no-install-recommends** | Option `apt-get install` excluant les paquets "recommandés" non strictement nécessaires, pour réduire la taille de l'image. |
| **--virtual (apk)** | Option `apk add` regroupant des paquets sous un méta-paquet nommé, supprimable en une seule commande (`apk del`). |
| **Cache mount (--mount=type=cache)** | Cache BuildKit persistant entre builds, monté uniquement pendant un `RUN`, jamais inclus dans la couche ou l'image finale. |
| **Probes Kubernetes (readiness/liveness/startup)** | Équivalent Kubernetes du `HEALTHCHECK` Dockerfile, défini dans le manifeste du Pod — Kubernetes ignore le `HEALTHCHECK` de l'image. |
| **Renovate / Dependabot** | Outils qui détectent automatiquement les nouvelles versions d'une dépendance ou d'une image et ouvrent une pull request de mise à jour. |
| **Content-Addressable Storage (CAS)** | Mécanisme identifiant chaque couche par le hash SHA256 de son contenu plutôt que par son nom, permettant la déduplication automatique entre images. |
| **manifest.json** | Fichier décrivant la composition d'une image exportée (configuration, liste des couches, tags), au format OCI Image Layout. |
| **Blob (image OCI)** | Fichier binaire nommé par le hash SHA256 de son contenu, représentant une couche, un manifest ou une configuration dans le format OCI. |
| **docker events** | Commande diffusant en temps réel les événements Docker (créations, démarrages, arrêts, suppressions) sur conteneurs, images, volumes et réseaux. |
| **OOM-kill** | Terminaison forcée du processus principal d'un conteneur par le noyau Linux lorsqu'il dépasse la limite `--memory` fixée. |
| **--cap-drop / --cap-add** | Flags `docker run` retirant ou ajoutant individuellement des capacités Linux à un conteneur, indépendamment du fait qu'il tourne en root ou non. |
| **--security-opt** | Flag `docker run` appliquant un module de sécurité supplémentaire (profil AppArmor, label SELinux, `no-new-privileges`) au conteneur. |
| **Docker Content Trust** | Mécanisme de vérification de signature d'image (`DOCKER_CONTENT_TRUST=1`) empêchant de puller une image dont l'origine ne peut pas être authentifiée. |
| **macvlan** | Driver réseau attribuant une adresse MAC/IP virtuelle à un conteneur, le rendant visible directement sur le réseau physique de l'hôte. |
| **docker system prune** | Commande supprimant en une fois les conteneurs arrêtés, réseaux et images non référencés, ainsi que le cache de build — sans toucher aux volumes par défaut. |
| **daemon.json** | Fichier de configuration du démon Docker (`/etc/docker/daemon.json`) appliquant certains réglages (rotation des logs, `icc`, `no-new-privileges`) à tous les conteneurs par défaut. |
| **Exit code** | Code numérique retourné par un conteneur à son arrêt ; au-delà de 128, indique un arrêt par signal (`code - 128` = numéro du signal). |
| **--init** | Flag `docker run` insérant un mini-init (`tini`) comme PID 1, qui relaie correctement les signaux à l'application même si son propre script d'entrée ne le fait pas via `exec`. |
| **docker update** | Commande modifiant certains réglages (`--restart`, `--memory`...) d'un conteneur existant sans le recréer. |
| **docker debug** | Commande Docker Desktop (licence Pro/Team/Business) attachant un shell de diagnostic externe à un conteneur ou une image dépourvus de shell (distroless, scratch). |
| **netshoot** | Image `nicolaka/netshoot` regroupant des outils de diagnostic réseau (`ping`, `curl`, `dig`, `tcpdump`), utile pour déboguer un conteneur applicatif minimal qui n'en dispose pas. |
| **Namespace (Linux)** | Mécanisme noyau isolant la vue qu'un processus a d'une ressource système (PID, NET, MNT, UTS, IPC, USER, CGROUP). |
| **User namespace / rootless** | Mode où l'UID 0 (root) du conteneur est mappé vers un utilisateur non privilégié de l'hôte — `userns-remap` (démon root) ou mode rootless (démon non-root). |
| **Cgroups v1 / v2** | Deux versions du mécanisme de limitation de ressources Linux ; la v2 (hiérarchie unifiée) est le standard actuel, la v1 est en mode maintenance depuis Kubernetes 1.31. |
| **--pids-limit** | Flag `docker run` limitant le nombre de processus qu'un conteneur peut créer, protection contre un fork bomb. |
| **Seccomp** | *Secure Computing Mode* — mécanisme noyau filtrant les appels système qu'un processus peut effectuer, indépendamment des capabilities. |
| **Profil seccomp** | Fichier JSON listant les syscalls autorisés (`SCMP_ACT_ALLOW`) ; tout syscall absent est refusé (`SCMP_ACT_ERRNO`) par défaut. |
| **capsh** | Outil (`libcap-ng-utils`) décodant un masque de capabilities hexadécimal (`CapEff`) en noms lisibles (`CAP_CHOWN`, `CAP_SYS_ADMIN`...). |
| **VXLAN** | Protocole d'encapsulation du trafic conteneur dans des paquets UDP, utilisé par le driver réseau `overlay` pour relier des conteneurs sur des hôtes différents. |
| **Macvlan** | Driver réseau attribuant une adresse MAC propre à chaque conteneur, qui apparaît alors comme un périphérique physique distinct sur le réseau local. |
| **Ipvlan** | Driver réseau proche de macvlan, mais où tous les conteneurs partagent l'adresse MAC de l'hôte ; supporte un mode L3 (routage) que macvlan ne supporte pas. |
| **enable_icc** | Option de driver réseau (`--opt com.docker.network.bridge.enable_icc=false`) désactivant la communication entre conteneurs sur un bridge donné. |
| **Réseau attachable (overlay)** | Option (`--attachable`) autorisant un conteneur standalone (hors service Swarm) à rejoindre un réseau overlay. |
| **Volume anonyme** | Volume créé sans nom explicite (`-v /chemin`), identifié par un hash aléatoire ; supprimé par `docker rm -v` ou `docker volume prune`. |
| **--mount (volume-nocopy)** | Option empêchant Docker de copier le contenu existant du conteneur vers un volume à sa création. |
| **Volume réseau (NFS/CIFS)** | Volume dont les données résident sur un serveur distant plutôt que sur le disque de l'hôte, via le driver `local` avec `--opt type=nfs`/`cifs`. |
| **Plugin de volume** | Extension Docker ajoutant un backend de stockage non natif (ex. `vieux/sshfs` pour monter un système de fichiers distant via SSH). |
| **Storage driver** | Mécanisme implémentant l'union filesystem et le copy-on-write sur le système de fichiers réel de l'hôte (`overlay2`, `btrfs`, `zfs`...). |
| **live-restore** | Option `daemon.json` maintenant les conteneurs en fonctionnement pendant un redémarrage du démon Docker. |
| **data-root** | Option `daemon.json` déplaçant l'emplacement de stockage des images/conteneurs/volumes hors de `/var/lib/docker`. |
| **Registry mirror** | Cache local (`registry-mirrors`) interrogé avant Docker Hub, pour réduire la bande passante et les rate limits. |
| **insecure-registries** | Option `daemon.json` désactivant la vérification TLS pour des registries précis — à réserver à un réseau strictement contrôlé. |
| **Credential helper** | Mécanisme délégant le stockage des identifiants de `docker login` au gestionnaire d'identifiants d'un fournisseur cloud, plutôt qu'à `~/.docker/config.json` en clair. |
| **Reload (SIGHUP) vs restart** | Deux façons d'appliquer `daemon.json` : certaines options s'appliquent à chaud sans couper les conteneurs, d'autres exigent un redémarrage complet du démon. |
| **--privileged** | Flag `docker run` désactivant toutes les protections (capacités, seccomp, AppArmor/SELinux) — à réserver à un usage isolé et temporaire, jamais à la production. |
| **Mode rootless** | Le démon Docker lui-même tourne sans privilèges root, pas seulement les conteneurs (contrairement à `userns-remap`). |
| **userns-remap** | Réglage `daemon.json` mappant l'UID de tous les conteneurs vers une plage non privilégiée de l'hôte, tout en gardant un démon root. |
| **`:z` / `:Z` (volume SELinux)** | Suffixes de montage ajustant le contexte SELinux d'un volume — `:z` partagé entre conteneurs, `:Z` exclusif à un seul, au risque de bloquer l'accès des autres. |
| **Trivy** | Scanner de vulnérabilités (CVE) dans les paquets d'une image Docker déjà construite. |
| **cosign / Sigstore** | Outil de signature et vérification *keyless* d'images, basé sur une identité OIDC plutôt qu'une clé privée gérée manuellement. |
| **Docker Bench Security** | Script officiel auditant la conformité au CIS Docker Benchmark sur l'hôte, le démon et les conteneurs en cours d'exécution. |
| **docker-socket-proxy** | Conteneur qui se place devant le vrai socket Docker, expose un endpoint TCP filtré, et refuse par défaut tout `POST` (écriture) — seuls les groupes d'endpoints explicitement déclarés sont ouverts. |
| **Groupe d'endpoints (socket-proxy)** | Ensemble de routes API activées par une variable du proxy (`CONTAINERS`, `IMAGES`, `NETWORKS`, `EVENTS`...), correspondant à un besoin de lecture précis. |
| **Docker Engine** | Le moteur Docker seul (démon + CLI), sans interface graphique — installation native sur Linux. |
| **Docker Desktop** | Distribution de Docker avec interface graphique pour Windows/macOS/Linux, gratuite en usage personnel, payante au-delà de 250 salariés ou 10M$ de revenus. |
| **Colima** | Alternative open source et gratuite à Docker Desktop sur macOS (*Containers on Lima*), sans interface graphique. |
| **docker.io vs docker-ce** | `docker.io` est le paquet packagé par les dépôts Ubuntu/Debian (souvent en retard) ; `docker-ce` vient du dépôt officiel Docker — à ne jamais installer ensemble. |
| **Groupe docker** | Groupe Unix dont l'appartenance permet d'utiliser Docker sans `sudo` — équivalent fonctionnel à un accès root sur la machine, comme le socket Docker. |
