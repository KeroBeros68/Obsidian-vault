#devops #docker #dockerfile

## Le Dockerfile

Un Dockerfile est un fichier texte contenant une suite d'instructions. Chaque instruction crée une nouvelle couche d'image, exécutée dans l'ordre du fichier.

```dockerfile
FROM python:3.12-slim      # image de base
WORKDIR /app                # dossier de travail dans l'image
COPY requirements.txt .     # copie un fichier précis
RUN pip install -r requirements.txt --no-cache-dir
COPY . .                    # copie le reste du code
CMD ["python", "app.py"]    # commande lancée au démarrage du conteneur
```

## Cache de build : l'ordre compte

Docker compare chaque instruction à son historique de build. Si une instruction et ses entrées sont identiques à une couche en cache, Docker réutilise le cache. Mais si une instruction change, Docker invalide le cache de cette instruction et de toutes celles qui suivent — elles devront être reconstruites.

| Situation | Syntaxe / Approche |
|-----------|-------------------|
| Dépendances qui changent rarement | Les `COPY` + `RUN install` en premier |
| Code source qui change souvent | Le `COPY . .` du code en dernier |
| Fichiers à exclure du contexte de build | `.dockerignore` |

```dockerfile
# ❌ Mauvais ordre : un changement de code invalide aussi l'install des deps
COPY . .
RUN pip install -r requirements.txt

# ✅ Bon ordre : les deps restent en cache si requirements.txt n'a pas changé
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

## Multi-stage build

Une seule image de build (avec compilateurs, outils, dépendances de dev) sert à fabriquer l'artefact ; une image finale minimale ne récupère que ce dont l'exécution a besoin.

```dockerfile
# Stage 1 — build
FROM golang:1.22-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server ./cmd/server

# Stage 2 — runtime minimal
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

Le binaire final ne transporte ni le compilateur Go, ni le code source — seulement l'exécutable.

## Cas particuliers

> [!warning] COPY . . trop tôt casse le cache
> Copier tout le code source avant d'installer les dépendances force Docker à réinstaller les dépendances à chaque changement de code, même mineur.

> [!tip] COPY plutôt qu'ADD
> `COPY` ne fait qu'une seule chose : copier des fichiers. `ADD` ajoute des comportements implicites (extraction d'archives, téléchargement d'URL) qui nuisent à la lisibilité. Préférer `COPY` sauf besoin explicite d'extraction d'archive locale.

> [!info] BuildKit
> Les fonctionnalités avancées de cache (`--mount=type=cache`) nécessitent BuildKit, activé par défaut sur les versions récentes de Docker.
