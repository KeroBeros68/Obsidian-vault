#devops #docker #sécurité #avancé

## Le problème : root par défaut

Par défaut, tout processus lancé dans un conteneur Docker s'exécute en tant qu'utilisateur `root` (UID 0) — y compris l'application elle-même. Si cette application est compromise (faille de code, dépendance vulnérable), l'attaquant hérite directement de privilèges root **dans le conteneur**, avec un risque réel d'évasion vers l'hôte selon la configuration (capacités Linux laissées actives, montages sensibles, faille du runtime).

C'est considérée comme la mauvaise configuration la plus fréquente en audit de sécurité de conteneurs — pas un raffinement optionnel réservé aux experts.

## Créer un utilisateur non-root

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Créer un groupe et un utilisateur dédiés, avec UID/GID explicites
RUN groupadd --gid 1000 appgroup && \
    useradd --uid 1000 --gid appgroup --shell /bin/false appuser

# Opérations nécessitant root (installation de paquets) AVANT de changer d'utilisateur
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code en attribuant directement la bonne propriété
COPY --chown=appuser:appgroup . .

# Toutes les instructions suivantes (RUN, CMD, ENTRYPOINT) s'exécutent en non-root
USER appuser

CMD ["python", "app.py"]
```

L'instruction `USER` doit venir **après** toutes les opérations qui nécessitent des privilèges root (installation de paquets système, par exemple) — une fois l'utilisateur changé, les permissions root ne sont plus disponibles pour le reste du Dockerfile.

## Utilisateurs non-root déjà fournis par certaines images

Certaines images officielles embarquent déjà un utilisateur non-root prêt à l'emploi, sans passer par `useradd`/`adduser` :

| Image | Utilisateur intégré | Utilisation |
|-------|----------------------|-------------|
| `node:*` | `node` (UID 1000) | `USER node` |
| `nginx:*` | `nginx` | `USER nginx` |
| `python:*` | — (aucun) | Créer manuellement, comme dans l'exemple ci-dessus |

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY --chown=node:node . .
USER node   # ✅ pas besoin de créer l'utilisateur, il existe déjà dans l'image
CMD ["node", "index.js"]
```

> [!tip] Vérifier avant de recréer un utilisateur
> Avant d'ajouter un `RUN useradd ...`, vérifier si l'image de base fournit déjà un utilisateur adapté (documentation de l'image, ou `docker run --rm image cat /etc/passwd`) évite une couche inutile et garantit un UID cohérent avec celui attendu par l'image elle-même.

## Renforcer au runtime

```bash
# Forcer un utilisateur précis, même si l'image en définit un autre
docker run --user 1000:1000 myapp:latest

# Système de fichiers racine en lecture seule (défense en profondeur)
docker run --read-only --tmpfs /tmp myapp:latest
```

Le flag `--user` au lancement écrase ce que le Dockerfile a défini avec `USER` — un second niveau de protection indépendant de l'image elle-même.

## Comportement et subtilités

| Situation | Approche |
|-----------|----------|
| Application qui doit écrire des fichiers (logs, cache) | Créer les dossiers et leur attribuer la propriété **avant** `USER` |
| UID cohérent entre plusieurs images d'une même stack | Fixer un UID numérique explicite (`--uid 1000`), pas un nom auto-généré |
| Port < 1024 à exposer (ex. port 80) | Nécessite `CAP_NET_BIND_SERVICE` même en non-root — préférer un port haut (ex. 8080) en interne |

```dockerfile
# ❌ Copie en root puis changement de propriétaire : couche supplémentaire inutile
COPY . .
RUN chown -R appuser:appgroup .
USER appuser

# ✅ Propriété attribuée directement à la copie
COPY --chown=appuser:appgroup . .
USER appuser
```

## --privileged : à ne jamais utiliser en production

```bash
# ❌ INTERDIT en production
docker run --privileged nginx
```

`--privileged` désactive **toutes** les protections vues dans ce module et dans [[Docker 11 — Sous le capot (namespaces, cgroups, seccomp)]] en une seule option : accès à tous les devices de l'hôte, toutes les capacités Linux réattribuées, aucun profil seccomp ni AppArmor, possibilité de charger des modules noyau ou de modifier les règles `iptables` de l'hôte. Un conteneur `--privileged` peut s'évader de son isolation en quelques commandes.

| Besoin qui pousse vers `--privileged` | Alternative sécurisée |
|------------------------------------------|---------------------------|
| Accéder à un device précis | `--device /dev/xxx` |
| Une capacité Linux spécifique | `--cap-add CAP_XXX` (voir plus bas) |
| Écouter sur un port < 1024 | `--cap-add NET_BIND_SERVICE` |
| Docker-in-Docker (build d'images en CI) | Runtime dédié (Sysbox) ou Podman, plutôt que `--privileged` |

> [!warning] Le seul cas d'usage légitime reste isolé et temporaire
> `--privileged` peut se justifier pour un environnement de test jetable et isolé du réseau (ex. certains setups Docker-in-Docker), jamais pour un conteneur applicatif exposé ou en production.

## Limiter les capacités Linux

Un conteneur root hérite par défaut d'un sous-ensemble de **capacités** Linux (privilèges root granulaires), pas de l'intégralité des droits root de l'hôte — mais ce sous-ensemble par défaut reste plus large que ce dont la plupart des applications ont réellement besoin.

```bash
# Retirer toutes les capacités, puis ne réattribuer que celle strictement nécessaire
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp:latest
```

`--cap-drop=ALL` retire l'ensemble des capacités par défaut ; `--cap-add` en rajoute une précise si l'application en a réellement besoin (ex. `NET_BIND_SERVICE` pour écouter sur un port < 1024). Combiné à un utilisateur non-root, cela réduit encore la marge de manœuvre d'un processus compromis, y compris s'il obtenait un accès root à l'intérieur du conteneur.

## Modules de sécurité additionnels (AppArmor / SELinux)

```bash
# AppArmor (Ubuntu/Debian) — applique un profil de confinement au conteneur
docker run --security-opt apparmor=docker-default nginx

# SELinux (RHEL/CentOS/Fedora) — applique un label de contexte de sécurité
docker run --security-opt label=type:svirt_apache_t nginx

# Bloquer toute élévation de privilège au sein du conteneur
docker run --security-opt no-new-privileges myapp:latest
```

Ces modules du noyau (*Linux Security Modules*) imposent des règles de confinement supplémentaires, indépendantes des capacités : même avec les bonnes capacités, un processus reste contraint par le profil AppArmor ou le label SELinux actif. `no-new-privileges` empêche un processus d'obtenir plus de privilèges qu'il n'en avait au démarrage (ex. via un binaire `setuid`), quelle que soit sa capacité initiale.

### Écrire un profil AppArmor personnalisé

Le profil `docker-default` appliqué automatiquement (Ubuntu/Debian) reste générique. Pour une application dont le comportement de fichiers est connu et stable, un profil dédié restreint bien plus précisément :

```
# /etc/apparmor.d/docker-nginx
#include <tunables/global>

profile docker-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  /var/www/** r,
  /etc/nginx/** r,
  /var/log/nginx/** rw,
  /var/run/nginx.pid rw,
  /var/run/nginx/*.sock rw,

  deny /proc/** w,
  deny /sys/** w,
}
```

```bash
sudo apparmor_parser -r /etc/apparmor.d/docker-nginx
docker run --security-opt apparmor=docker-nginx nginx
```

### Configurer SELinux pour Docker

```json
{ "selinux-enabled": true }
```

```bash
getenforce   # Enforcing, Permissive, ou Disabled
docker inspect --format '{{.ProcessLabel}}' my-container
```

Un volume monté hérite d'un contexte SELinux qui détermine si le conteneur peut y accéder :

```bash
docker run -v /data:/data:z myapp   # :z — contexte partagé, plusieurs conteneurs peuvent y accéder
docker run -v /data:/data:Z myapp   # :Z — contexte privé, ce conteneur uniquement
```

> [!warning] `:Z` peut casser l'accès d'autres services au même répertoire
> `:Z` réétiquette le contexte SELinux du répertoire hôte pour le réserver à ce conteneur — si un autre processus (un autre conteneur, un service système) accède au même chemin, il perd l'accès après ce réétiquetage. Préférer `:z` (partagé) sauf besoin explicite d'exclusivité.

## Confiance des images : Docker Content Trust

```bash
# Activer la vérification de signature avant tout pull
export DOCKER_CONTENT_TRUST=1
docker pull nginx   # échoue si l'image n'est pas signée
```

Avec `DOCKER_CONTENT_TRUST=1`, Docker refuse de puller une image dont la signature ne peut pas être vérifiée — une protection contre la substitution d'une image par une version compromise sur le registry, au-delà du simple contrôle d'accès.

```bash
docker trust inspect nginx:latest        # voir les signatures existantes d'une image
docker trust sign myregistry/myapp:v1.0  # signer une image (nécessite une clé de signature)
```

> [!info] cosign, une alternative moderne à Content Trust
> Le projet **Sigstore/cosign** couvre un besoin proche avec un mécanisme différent (attestations SLSA, signatures *keyless* liées à une identité OIDC plutôt qu'à une clé gérée manuellement) — voir [[Docker 09 — Outils d'analyse & linting]] pour son usage.

> [!info] Content Trust ne remplace pas le scan de vulnérabilités
> Une image signée garantit qu'elle provient bien de l'éditeur attendu et n'a pas été altérée — elle ne garantit pas l'absence de vulnérabilités connues dans son contenu. Les deux contrôles sont complémentaires : signature d'origine (Content Trust) et scan de CVE (Trivy, voir [[Docker 09 — Outils d'analyse & linting]]).

## Le socket Docker : un accès root déguisé

```yaml
services:
  ci-runner:
    image: some-ci-tool
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock   # ⚠️ équivaut à un accès root sur l'hôte
```

Le démon Docker s'exécute en root. Un conteneur qui a accès à `/var/run/docker.sock` peut demander au démon de lancer n'importe quel autre conteneur — y compris un conteneur `--privileged` avec `/` de l'hôte monté dedans. Concrètement, un accès au socket **équivaut à un accès root complet sur la machine hôte**, même si le conteneur qui le détient tourne lui-même en non-root en interne.

> [!warning] Cas d'usage légitimes, mais à isoler
> Certains outils ont un besoin réel du socket (reverse-proxy qui découvre les conteneurs dynamiquement, dashboard d'administration, CI qui build des images). Dans ce cas : dédier un conteneur unique à ce rôle, ne jamais lui donner de rôle applicatif en plus, et envisager un proxy socket en lecture seule qui filtre les commandes autorisées plutôt que d'exposer le socket brut.

> [!tip] `:ro` ne suffit pas à sécuriser le socket
> Monter le socket en lecture seule (`:ro`) n'empêche pas d'émettre des commandes de création ou d'arrêt de conteneur — l'API Docker fonctionne par écriture sur le socket lui-même, et `:ro` ne porte que sur le fichier, pas sur le contenu des requêtes. Démonstration concrète et déploiement d'un `docker-socket-proxy` filtrant dans [[Docker 15 — Socket-proxy (sécuriser l'accès au socket)]].

## Cas particuliers

> [!warning] USER puis retour à root casse la logique
> Remettre `USER root` après un premier `USER appuser` pour exécuter une dernière opération privilégiée, puis repasser en non-root, fonctionne mais fragilise la lisibilité et le contrôle du Dockerfile. Toujours regrouper les opérations root en un seul bloc, avant le `USER` final.

> [!tip] UID numérique plutôt que nom
> Préférer un UID explicite (`1000`) à un nom d'utilisateur pour la compatibilité avec les contextes de sécurité d'orchestrateurs externes (ex. Kubernetes `runAsNonRoot`), qui raisonnent en UID plutôt qu'en nom.

> [!info] Defense in depth
> L'utilisateur non-root est une couche de protection parmi d'autres — combiné à `--read-only`, à la suppression des capacités Linux inutiles (`--cap-drop=ALL`), et à des images de base minimales (slim, alpine, distroless), il réduit significativement la surface d'attaque sans la supprimer entièrement.

> [!info] Vérifier ces pratiques automatiquement
> Dockle et Checkov détectent automatiquement l'absence de `USER` non-root et d'autres écarts de sécurité sur une image ou un Dockerfile, en s'appuyant sur les CIS Docker Benchmarks. Voir [[Docker 09 — Outils d'analyse & linting]].

## Dépannage sécurité

Un durcissement trop agressif se traduit souvent par ces symptômes plutôt que par un message d'erreur explicite :

| Symptôme | Cause probable | Solution |
|----------|------------------|----------|
| `permission denied` dans le conteneur | Filesystem en `--read-only` sans `tmpfs` pour l'écriture attendue | Ajouter `--tmpfs /chemin` pour les fichiers temporaires nécessaires |
| `operation not permitted` | Capacité Linux manquante après `--cap-drop=ALL` | `--cap-add` la capacité précise requise, pas `ALL` |
| Conteneur tué de façon répétée (exit 137) | Limite `--memory` trop basse pour l'application | Augmenter la limite ou optimiser la consommation — voir [[Docker 02 — Cycle de vie & debugging]] |
| Échec de bind sur un port | Mode rootless + port < 1024 | Utiliser un port > 1024, ou une capacité adaptée — voir [[Docker 11 — Sous le capot (namespaces, cgroups, seccomp)]] |
| `Volume permission denied` | User namespace actif (`userns-remap`) | Ajuster les UID/GID du volume, ou `--userns=host` si le conteneur doit accéder à des fichiers existants avec leur propriétaire d'origine |

> [!info] Application concrète : confiner du code généré par un LLM
> Ces options combinées (`--network none`, `--cap-drop ALL`, `--read-only`, `--user` non-root) forment la base d'un bac à sable pour exécuter du code potentiellement hostile — c'est exactement le cas d'un agent IA qui exécute du code écrit par un modèle. Voir [[Agents 10 — Sandboxing du code généré par un LLM]] pour cette application, avec un conteneur jetable et un délai appliqué côté client.
