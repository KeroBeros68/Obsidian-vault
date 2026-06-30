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

## Cas particuliers

> [!warning] USER puis retour à root casse la logique
> Remettre `USER root` après un premier `USER appuser` pour exécuter une dernière opération privilégiée, puis repasser en non-root, fonctionne mais fragilise la lisibilité et le contrôle du Dockerfile. Toujours regrouper les opérations root en un seul bloc, avant le `USER` final.

> [!tip] UID numérique plutôt que nom
> Préférer un UID explicite (`1000`) à un nom d'utilisateur pour la compatibilité avec les contextes de sécurité d'orchestrateurs externes (ex. Kubernetes `runAsNonRoot`), qui raisonnent en UID plutôt qu'en nom.

> [!info] Defense in depth
> L'utilisateur non-root est une couche de protection parmi d'autres — combiné à `--read-only`, à la suppression des capacités Linux inutiles (`--cap-drop=ALL`), et à des images de base minimales (slim, alpine, distroless), il réduit significativement la surface d'attaque sans la supprimer entièrement.
