#devops #secrets #docker

## Le contexte de build peut fuiter avant même le premier COPY

`docker build` envoie tout le contenu du dossier de contexte au démon Docker avant d'exécuter la moindre instruction. Sans `.dockerignore`, un `.env` ou un dossier `secrets/` présents à côté du Dockerfile peuvent être copiés par inadvertance via un `COPY . .` trop large.

```
# .dockerignore
.env
.env.*
secrets/
*.pem
*.key
.git
```

Un `!` en début de ligne réintroduit une exception dans un motif d'exclusion plus large :

```
# .dockerignore
*.md
!README*.md
README-secret.md
```

Ici, tous les fichiers `.md` sont exclus, sauf ceux commençant par `README` — à l'exception de `README-secret.md`, ré-exclu explicitement après. L'ordre des lignes compte : une exception ne peut réintroduire que ce qu'un motif précédent a exclu.

> [!tip] Vérifier ce qui part réellement au build
> La première ligne affichée par `docker build` indique la taille du contexte envoyé au démon — un moyen rapide de vérifier qu'un `.dockerignore` incomplet ne laisse pas fuiter des fichiers volumineux ou sensibles.
> ```bash
> docker build . 2>&1 | head -1
> # "Sending build context to Docker daemon  2.048kB"  ← attendu
> # "Sending build context to Docker daemon  500MB"     ← .dockerignore incomplet
> ```

> [!info] Voir aussi
> Le rôle du cache de build et l'ordre des instructions `COPY` sont couverts dans [[Docker 03 — Dockerfile]].

## ARG et ENV : le piège de `docker history`

`ARG` et `ENV` finissent tous les deux dans les métadonnées de l'image, consultables sans avoir besoin de démarrer le conteneur :

```dockerfile
ARG NPM_TOKEN
RUN npm install   # utilise $NPM_TOKEN pour s'authentifier sur un registry privé
```

```bash
docker history --no-trunc myimage
# révèle la valeur de NPM_TOKEN passée au build, même si aucune instruction
# ultérieure ne la réutilise explicitement
```

> [!warning] Ni ARG ni ENV ne conviennent pour un secret de build
> Une valeur passée par `--build-arg` ou fixée par `ENV` reste lisible dans l'historique de l'image indéfiniment, y compris après un `docker push` vers un registry public ou privé. Ce n'est pas un détail d'affichage local : n'importe qui avec un accès `docker pull` peut l'extraire.

## `RUN --mount=type=secret` (BuildKit)

BuildKit fournit un mécanisme dédié : le secret est monté uniquement pendant l'exécution d'un `RUN`, et n'est jamais écrit dans la couche résultante ni dans les métadonnées de l'image.

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install
```

```bash
docker build --secret id=npm_token,src=./npm_token.txt -t myapp .
```

| Mécanisme | Persiste dans l'image finale | Visible via `docker history` |
|------------|-------------------------------|-------------------------------|
| `ARG` / `ENV` | ❌ non (sauf s'il finit dans un fichier copié) | ✅ oui |
| `RUN --mount=type=secret` | ❌ non | ❌ non |

### Options avancées du mount secret

| Option | Rôle |
|--------|------|
| `id` | Identifiant du secret (obligatoire, référencé par `docker build --secret id=...`) |
| `target` | Chemin de montage dans le conteneur (défaut : `/run/secrets/<id>`) |
| `required=true` | Fait échouer le build si le secret n'est pas fourni, plutôt que de continuer silencieusement sans lui |
| `mode` | Permissions du fichier monté, en octal (ex. `mode=0400`) |
| `uid` / `gid` | Propriétaire du fichier monté |

```dockerfile
RUN --mount=type=secret,id=db_password,target=/run/secrets/db,required=true,mode=0400,uid=1000 \
    /app/init-db.sh
```

### Injecter un secret directement en variable d'environnement

```dockerfile
# Le secret est disponible comme $API_KEY, sans fichier intermédiaire
RUN --mount=type=secret,id=api_key,env=API_KEY \
    curl -H "Authorization: Bearer $API_KEY" https://api.example.com/data
```

> [!tip] Fichier plutôt que variable, même avec BuildKit
> L'option `env=` évite un fichier, mais réintroduit le risque propre aux variables d'environnement (fuite via `/proc/<pid>/environ`, logs d'erreur d'un outil, `env` accidentel — voir [[Secrets 02 — Variables d'environnement vs fichiers montés]]). Réserver `env=` aux cas où l'outil appelé n'accepte que ce format (ex. `curl -H "Authorization: Bearer $VAR"`), et préférer `target=` par défaut.

## Vérifier qu'un secret n'a pas fuité

```bash
# Rechercher des mots-clés révélateurs dans l'historique de l'image
docker history mon-image --no-trunc | grep -iE 'password|secret|token|key'

# Rechercher des fichiers suspects dans l'image construite
docker run --rm mon-image find / -name "*.env" -o -name "*secret*" 2>/dev/null

# Scanner spécifiquement les secrets (pas seulement les CVE) avec Trivy
trivy image mon-image --scanners secret
```

| Outil | Usage |
|-------|-------|
| Trivy (`--scanners secret`) | Scan de secrets dans une image déjà construite, en plus de son usage CVE classique (voir [[Docker 09 — Outils d'analyse & linting]]) |
| trufflehog | Détection de secrets dans un historique Git ou directement dans une image (`trufflehog docker --image ...`) |
| Gitleaks | Hook pre-commit, bloque un `git commit` qui introduirait un secret avant même qu'il n'atteigne le dépôt distant |

## Cas particuliers

> [!tip] `--mount=type=secret` est la réponse correcte à "j'ai besoin d'un token pendant le build"
> Chaque fois qu'un build a besoin d'un identifiant temporaire (token npm/pip privé, clé SSH pour cloner un dépôt privé), c'est le cas d'usage prévu pour ce mécanisme — pas `ARG`.

> [!warning] Nécessite BuildKit et la syntaxe `# syntax=`
> `--mount=type=secret` n'existe que sous BuildKit et requiert la ligne `# syntax=docker/dockerfile:1` en tête de Dockerfile pour activer la syntaxe étendue.
