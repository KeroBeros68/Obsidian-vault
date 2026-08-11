#ia #qdrant #installation #fondamentaux

## Lancer Qdrant avec Docker

L'image officielle expose son API sur le port `6333`.

```bash
docker run -d --name qdrant -p 6333:6333 qdrant/qdrant
```

```bash
curl http://localhost:6333/
# Doit renvoyer un JSON avec "title": "qdrant - vector search engine"
```

## Le client Python

```bash
pip install qdrant-client==1.18.0
```

```python
from qdrant_client import QdrantClient

client = QdrantClient(host="localhost", port=6333)
```

## Persister les données

La commande `docker run` ci-dessus garde les données **dans le conteneur** — les supprimer avec le conteneur. Pour un usage réel, monter un volume :

```bash
docker run -d --name qdrant -p 6333:6333 \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

> [!info] Le même principe que n'importe quel service conteneurisé avec état
> Sans volume monté sur `/qdrant/storage`, l'index disparaît à chaque suppression ou recréation du conteneur — le rôle du volume ici est identique à celui décrit pour n'importe quelle base de données conteneurisée (voir [[Docker 05 — Volumes & persistance]] si ce mécanisme n'est pas encore familier).

## Pour aller plus loin

Une fois Qdrant lancé et accessible, la première étape côté application est de créer une collection — voir [[Qdrant 03 — Collections & indexation avec payload]].

Sources : [Qdrant — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/qdrant/)
