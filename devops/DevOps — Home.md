#devops #home #index

## Modules disponibles

### Scripting & Automatisation

- [[Bash — Index des fiches]]
- [[Cron — Index des fiches]]

### Conteneurs

- [[Conteneurs — Généralités]] ← concepts communs (namespaces, cgroups, capabilities, panorama de l'écosystème)
- [[Conteneurs — Moteurs]] ← comparatif Docker/Podman/containerd/CRI-O/LXC/Incus, pour choisir son moteur
- [[Conteneurs — OCI]] ← standard OCI : tag/digest, manifest, multi-arch, artefacts
- [[Conteneurs — Glossaire]]
- [[Docker — Index des fiches]]
- [[Builders — Index des fiches]] ← outils de build d'images (BuildKit, Buildah, Kaniko, Buildpacks, Bake)

### Serveurs web

- [[Serveurs Web — Choisir son serveur]] ← Nginx vs Apache vs Caddy
- [[Serveurs Web — Concepts fondamentaux]] ← HTTP, virtual hosts, reverse proxy, TLS, cache, logs
- [[Serveurs Web — Checklist production & dépannage]]
- [[Serveurs Web — Glossaire]]
- [[Nginx — Index des fiches]]
- [[Apache — Index des fiches]]
- [[Caddy — Index des fiches]]

### Sécurité

- [[Secrets — Index des fiches]]

### Transfert de fichiers

- [[FTP — Index des fiches]] ← protocole de base : canaux contrôle/données, actif/passif, FTPS, distinction avec SFTP/SCP
- [[Pure-FTPd — Index des fiches]] ← configuration par fichiers dédiés, utilisateurs virtuels (PureDB), chroot, TLS, quotas

## Parcours recommandés

```
Scripting   : Bash 01-04 → 05-07 → 08-09 → Cron 01-04
Conteneurs  : Docker 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 15 → Builders 01 → 02
Serveurs web : Nginx 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 10 → 11 → 08 → 12 → 09 → 13 → 14 → 15 → 16, Apache 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13, Caddy 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 (prérequis : Serveurs Web — Choisir son serveur → Concepts fondamentaux)
Sécurité    : Docker 08 → 11 → Secrets 01 → 02 → 03 → 04 → 05 → 06
Transfert de fichiers : FTP 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07, Pure-FTPd 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 (prérequis : FTP — Index des fiches)
Déploiement : Bash → Docker → Nginx → FastAPI → Kubernetes (manquant)
```

## Prérequis & suite

- [[Home]] ← retour accueil
- [[Python — Home]] ← FastAPI, apps déployées dans les conteneurs
- [[Manques]] ← Kubernetes (P4, non couvert), CI/CD (P4, non couvert), implémentations serveur FTP restantes (vsftpd, ProFTPD — modules à venir)
