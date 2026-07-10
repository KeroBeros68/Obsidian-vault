#devops #builders #pièges #erreurs #debugging

## 🪤 Piège 1 — Confondre BuildKit et Docker

```
❌ "BuildKit" et "Docker" comme s'ils étaient deux alternatives concurrentes
✅ BuildKit est le builder utilisé PAR Docker (depuis Docker 18.09, par défaut depuis 23.0)
```

> [!tip] Mémo
> BuildKit est aussi utilisable en standalone (`buildkitd`), sans Docker du tout — mais dans un usage Docker classique, ce n'est pas un choix concurrent, c'est déjà ce qui tourne sous le capot. Voir [[Builders 01 — Qu'est-ce qu'un outil de build]].

---

## 🪤 Piège 2 — Oublier le cache en CI/CD

```bash
# ❌ Chaque run CI repart de zéro : aucun cache partagé entre machines
docker build -t mon-app .
```

```bash
# ✅ Cache partagé via registry, réutilisable d'un run à l'autre
docker buildx build \
  --cache-from type=registry,ref=ghcr.io/user/cache \
  --cache-to type=registry,ref=ghcr.io/user/cache,mode=max \
  -t mon-app .
```

> [!warning] Un runner CI éphémère n'a pas de cache local persistant
> Sans cache registry (ou équivalent `type=gha` pour GitHub Actions), chaque exécution repart d'un environnement vierge — le cache local de BuildKit ne survit pas d'un run à l'autre sur un runner jetable. Voir [[Builders 02 — Panorama des outils]].

---

## 🪤 Piège 3 — Croire que Buildpacks retire tout contrôle

```
❌ "Buildpacks = boîte noire, aucune personnalisation possible"
✅ Des variables d'environnement et des buildpacks personnalisés existent pour ajuster le comportement
```

> [!tip] Mémo
> Zéro configuration par défaut ne veut pas dire zéro configuration possible — les builders Buildpacks (ex. Paketo) exposent des variables d'environnement pour ajuster le comportement, et des buildpacks custom peuvent s'ajouter au processus de détection.

---

## 🪤 Piège 4 — Utiliser Kaniko sur un poste de développement local

```bash
# ⚠️ Fonctionne techniquement, mais lent et peu ergonomique en local
docker run -v $(pwd):/workspace gcr.io/kaniko-project/executor:latest \
  --context=/workspace --destination=mon-app:latest --no-push
```

> [!warning] Kaniko est pensé pour un conteneur, pas un poste de dev
> Kaniko est conçu pour tourner dans un pod Kubernetes ou un job CI, sans daemon Docker disponible. En local, BuildKit ou Buildah restent plus rapides et plus ergonomiques. Voir [[Builders 02 — Panorama des outils]].

---

## 🪤 Piège 5 — Tenter du multi-arch sans QEMU installé

```bash
# ❌ Échoue ou tombe en erreur d'exécution sur une plateforme non native
docker buildx build --platform linux/amd64,linux/arm64 -t mon-app .
```

```bash
# ✅ Enregistrer les handlers QEMU avant un build multi-arch sur Linux
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

> [!tip] Runners natifs plutôt que l'émulation, si la performance compte
> QEMU permet de builder pour une architecture différente de l'hôte, mais reste plus lent qu'une exécution native — pour un usage CI fréquent, des runners ARM64 physiques restent préférables à l'émulation systématique.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| BuildKit perçu comme concurrent de Docker | BuildKit est le builder intégré à Docker, pas une alternative |
| Cache absent en CI/CD | `--cache-from`/`--cache-to type=registry` (ou `type=gha`) |
| Buildpacks jugé sans contrôle possible | Variables d'environnement + buildpacks personnalisés |
| Kaniko utilisé en local | Réserver Kaniko à Kubernetes/CI, BuildKit ou Buildah en local |
| Multi-arch sans QEMU enregistré | Enregistrer `multiarch/qemu-user-static` avant le build |
