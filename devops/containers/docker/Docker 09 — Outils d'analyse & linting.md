#devops #docker #avancé #qualité

## Plusieurs outils, plusieurs angles différents

Écrire un Dockerfile qui fonctionne ne garantit ni qu'il soit optimisé, ni qu'il soit sécurisé. Ces outils analysent chacun un angle différent, en complément des bonnes pratiques déjà vues dans [[Docker 03 — Dockerfile]] et [[Docker 08 — Sécurité des conteneurs]].

| Outil | Analyse | Quand l'utiliser |
|-------|---------|-------------------|
| **Hadolint** | Le Dockerfile lui-même (syntaxe, bonnes pratiques) | Avant même de builder — un linter, comme ESLint pour le JS |
| **Dive** | Les couches de l'image construite | Après un build, pour repérer la taille et le gaspillage |
| **Dockle** | La conformité CIS de l'image construite | Après un build, en complément de Dive côté sécurité |
| **Trivy** | Les CVE connues dans les paquets de l'image | En CI, sur chaque image avant déploiement |
| **Docker Bench Security** | L'hôte, le démon et les conteneurs en exécution | Audit ponctuel, plus large qu'une seule image |
| **Checkov** | Le Dockerfile (et d'autres fichiers IaC : Terraform, Kubernetes...) | En CI, pour un contrôle de sécurité unifié multi-outils |

## Hadolint — linter de Dockerfile

Analyse le texte du Dockerfile avant tout build, avec des règles dédiées (préfixées `DL`) et une intégration de **ShellCheck** pour le contenu des instructions `RUN`.

```bash
hadolint Dockerfile
```

```
Dockerfile:3 DL3006 warning: Always tag the version of an image explicitly
Dockerfile:5 DL3009 info: Delete the apt-get lists after installing something
Dockerfile:8 DL3025 warning: Use arguments JSON notation for CMD and ENTRYPOINT
```

> [!tip] Règles désactivables par projet
> Un fichier `.hadolint.yaml` à la racine du projet permet d'ignorer certaines règles (`ignored: [DL3008]`) quand une contrainte du projet justifie une exception délibérée, plutôt que de désactiver le linter entièrement.

## Mesurer avant d'optimiser

Avant de sortir Dive, un premier diagnostic tient en deux commandes natives Docker, sans outil supplémentaire :

```bash
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}"   # taille de chaque image locale
docker history <image>:<tag>                                          # détail couche par couche
```

> [!tip] Objectif réaliste selon le langage
> Une réduction de 50% est un minimum atteignable dans la plupart des cas. Pour un binaire Go ou Rust compilé statiquement, viser moins de 10 Mo est réaliste (voir `scratch` dans [[Docker 03 — Dockerfile]]) ; pour Python ou Node.js, 50 à 150 Mo reste un bon objectif une fois les optimisations appliquées.

## Dive — exploration des couches et gaspillage

Dive va plus loin que `docker history` en offrant une vue interactive : il ouvre une image construite couche par couche, affiche un **score d'efficacité** et repère les fichiers dupliqués ou modifiés inutilement d'une couche à l'autre.

```bash
dive myapp:latest
```

En mode CI, `dive` peut faire échouer un pipeline si l'image dépasse des seuils définis :

```bash
dive myapp:latest --ci --highestUserWastedPercent 0.1 --lowestEfficiency 0.9
```

> [!info] Complète les fiches sur le choix d'image de base
> Un score d'efficacité bas confirme souvent un problème déjà couvert dans [[Docker 03 — Dockerfile]] (cache mal ordonné, `apt-get` non nettoyé dans la même couche) ou dans le choix de l'image de base (Debian complet plutôt que `-slim`/Alpine).

## Dockle — sécurité de l'image construite

Vérifie l'image finale par rapport aux **CIS Docker Benchmarks** (recommandations de sécurité standardisées) : utilisateur non-root, présence d'un `HEALTHCHECK`, absence de cache de gestionnaire de paquets non nettoyé, secrets potentiellement présents dans les couches.

```bash
dockle myapp:latest
```

```
WARN    - CIS-DI-0001: Create a user for the container
INFO    - CIS-DI-0005: Enable Content trust for Docker
```

Les niveaux vont de `PASS` à `FATAL`, avec `INFO`/`WARN` intermédiaires — utile pour prioriser les corrections plutôt que tout traiter comme bloquant d'emblée.

## Trivy — scanner de vulnérabilités de référence

Contrairement à Dockle (conformité CIS de l'image), Trivy scanne les **CVE connues** dans les paquets et dépendances d'une image.

```bash
trivy image nginx:latest
trivy image --severity HIGH,CRITICAL nginx:latest    # ignorer les sévérités mineures
trivy image --ignore-unfixed nginx:latest             # masquer les CVE sans correctif disponible (rien à faire dessus pour l'instant)
trivy image -f json -o results.json nginx:latest      # export pour un pipeline CI
```

```yaml
# GitHub Actions
- name: Scan image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'
```

| Alternative à Trivy | Particularité |
|------------------------|------------------|
| Grype (Anchore) | Léger, rapide |
| Snyk | Intégration IDE |
| Clair (Quay) | Scanner intégré au registry |
| Docker Scout | Intégré à Docker Desktop |

> [!tip] `--ignore-unfixed` évite de bloquer une CI sur l'incorrigible
> Une CVE sans correctif publié par le mainteneur du paquet ne peut pas être résolue immédiatement — la remonter comme un échec bloquant de pipeline n'aide personne. `--ignore-unfixed` la masque du rapport tant qu'aucun correctif n'existe, sans pour autant l'ignorer définitivement (elle réapparaît dès qu'un correctif sort).

## Vérifier la provenance : cosign / Sigstore

Alternative moderne à [[Docker 08 — Sécurité des conteneurs]] (Docker Content Trust), basée sur des attestations **SLSA** plutôt que sur une clé de signature classique :

```bash
cosign verify ghcr.io/sigstore/cosign:latest \
  --certificate-identity keyless@sigstore \
  --certificate-oidc-issuer https://accounts.google.com
```

La signature *keyless* lie l'image à une identité OIDC (compte GitHub, Google...) au moment de la publication, plutôt qu'à une clé privée à gérer et faire tourner manuellement.

## Docker Bench Security — audit CIS complet

Script officiel qui vérifie la conformité au **CIS Docker Benchmark** sur l'hôte, le démon et les conteneurs en cours d'exécution — plus large que Dockle, qui ne juge qu'une image isolée.

```bash
docker run --rm -it \
  --net host --pid host --userns host \
  --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /etc:/etc:ro \
  docker/docker-bench-security
```

| Section | Contenu vérifié |
|---------|--------------------|
| 1 | Configuration de l'hôte |
| 2 | Configuration du démon |
| 3 | Fichiers de configuration |
| 4 | Images et Dockerfiles |
| 5 | Runtime des conteneurs |
| 6 | Opérations de sécurité |
| 7 | Docker Swarm |

Parmi les dizaines de règles CIS, celles qui apportent le plus de gain pour l'effort investi :

| ID | Recommandation | Impact |
|----|-------------------|--------|
| 2.1 | Activer `live-restore` | Continuité de service (voir [[Docker 10 — Configuration production & nettoyage]]) |
| 2.2 | Désactiver `userland-proxy` | Performance + sécurité |
| 4.1 | Utilisateur non-root dans les images | Isolation (voir [[Docker 08 — Sécurité des conteneurs]]) |
| 5.1 | Ne jamais utiliser `--privileged` | Critique |
| 5.2 | Limiter les capacités (`--cap-drop=ALL`) | Moindre privilège |
| 5.10 | Limiter la mémoire (`--memory`) | Prévention DoS |

> [!info] Ce script requiert lui-même des accès larges à l'hôte
> Les montages (`--net host`, `--pid host`, `/etc:ro`...) sont nécessaires pour que l'audit puisse inspecter la configuration réelle de l'hôte et du démon — cohérent avec son rôle d'audit, mais à ne lancer que ponctuellement, pas comme un conteneur applicatif permanent.

## Checkov — scanner IaC unifié

Analyse aussi bien un Dockerfile qu'un fichier Terraform, Kubernetes ou CloudFormation avec le même moteur de règles — utile quand un projet mélange plusieurs formats d'infrastructure-as-code.

```bash
checkov -f Dockerfile
```

Pour un Dockerfile, Checkov vérifie entre autres : présence d'un `USER` non-root, présence d'un `HEALTHCHECK`, absence de tag `:latest`, et tente de détecter des secrets codés en dur — recoupant volontairement certains points déjà couverts par Hadolint et Dockle, mais dans un seul outil au périmètre plus large.

## Cas particuliers

> [!tip] Les combiner plutôt que choisir un seul outil
> Hadolint (avant build) + Dive (taille) + Dockle (sécurité de l'image) ciblent des défauts différents et se recoupent peu ; les faire tourner tous les trois en CI coûte peu et couvre plus de terrain qu'un seul outil. Checkov est une alternative pertinente surtout si le projet a déjà d'autres fichiers IaC à scanner avec le même outil.

> [!info] Aucun de ces outils ne remplace une revue humaine
> Ces linters détectent des écarts par rapport à des règles connues et documentées (cache apt, utilisateur root, tag `latest`...) — ils ne remplacent pas une réflexion sur l'architecture de l'image ou les besoins réels de sécurité d'un déploiement précis.
