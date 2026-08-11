#ia #qdrant #fondamentaux

## Créer une collection

Une **collection** est l'unité de stockage de Qdrant — l'équivalent d'une table. Elle est typée à sa création par deux paramètres : la dimension des vecteurs et la distance de comparaison.

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

DIMENSION = 768  # taille des vecteurs de nomic-embed-text
COLLECTION = "documentation"

client = QdrantClient(host="localhost", port=6333)

if client.collection_exists(COLLECTION):
    client.delete_collection(COLLECTION)

client.create_collection(
    COLLECTION,
    vectors_config=VectorParams(size=DIMENSION, distance=Distance.COSINE),
)
```

> [!warning] La dimension doit correspondre exactement au modèle d'embedding
> `size` doit correspondre exactement à la dimension produite par le modèle d'embedding utilisé (768 pour `nomic-embed-text`, voir [[RAG 02 — Embeddings et Vector Databases]]) — un vecteur d'une autre taille est rejeté à l'écriture. `distance=Distance.COSINE` est le choix standard pour des vecteurs de texte, la même mesure que la similarité cosinus détaillée dans le module RAG.

## Le point : identifiant, vecteur, payload

Dans Qdrant, l'unité indexée est le **point**, qui réunit trois éléments : un identifiant, un vecteur, et un **payload**.

```python
from qdrant_client.models import PointStruct

CORPUS = [
    {"texte": "Un volume Docker conserve les données du conteneur.",
     "sujet": "docker", "annee": 2026},
    {"texte": "Terraform décrit une infrastructure en code déclaratif.",
     "sujet": "terraform", "annee": 2024},
]

vecteurs = vectoriser([d["texte"] for d in CORPUS])

points = [
    PointStruct(id=i, vector=vecteurs[i], payload=CORPUS[i])
    for i in range(len(CORPUS))
]

client.upsert(COLLECTION, points=points)
```

> [!info] Le payload est l'atout de Qdrant
> Le payload est un dictionnaire libre attaché au point : le texte d'origine, mais aussi toute métadonnée utile (un sujet, une année, une équipe propriétaire). Ce payload voyage avec le vecteur et devient **filtrable** à la recherche — voir [[Qdrant 04 — Rechercher & filtrer par payload]].

`upsert` insère les points, ou les met à jour s'ils existent déjà — d'où son nom (*update* + *insert*).

## Pour aller plus loin

Une fois les points indexés, la recherche sémantique — avec ou sans filtre sur le payload — est couverte dans [[Qdrant 04 — Rechercher & filtrer par payload]].

Sources : [Qdrant — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/qdrant/)
