#ia #chroma #recherche #filtres #metadata #pratique

## Recherche vectorielle et filtres

La puissance de Chroma vient de sa capacité à combiner la recherche vectorielle (sens) avec des filtres sur les métadonnées (critères exacts).

## query() — recherche vectorielle de base

```python
# Recherche par texte (embedding calculé automatiquement)
résultats = collection.query(
    query_texts=["politique de retour produit"],
    n_results=4,
    include=["documents", "metadatas", "distances"]
)

# Recherche par vecteur pré-calculé
embedding_requête = model.encode("politique de retour produit").tolist()
résultats = collection.query(
    query_embeddings=[embedding_requête],
    n_results=4
)

# Plusieurs requêtes en parallèle
résultats = collection.query(
    query_texts=["retour produit", "livraison express", "garantie"],
    n_results=2
)
# résultats["documents"][0] → top-2 pour "retour produit"
# résultats["documents"][1] → top-2 pour "livraison express"
# résultats["documents"][2] → top-2 pour "garantie"
```

## Filtres sur les métadonnées — where

```python
# ── Filtres simples ───────────────────────────────────────

# Égalité
résultats = collection.query(
    query_texts=["retour"],
    n_results=3,
    where={"source": "politique.txt"}   # seulement ce fichier
)

# Opérateurs de comparaison
résultats = collection.query(
    query_texts=["erreur"],
    n_results=3,
    where={"page": {"$gte": 5}}    # page >= 5
    # Opérateurs : $eq, $ne, $gt, $gte, $lt, $lte, $in, $nin
)

# ── Filtres composés ──────────────────────────────────────

# ET logique
résultats = collection.query(
    query_texts=["configuration"],
    n_results=3,
    where={
        "$and": [
            {"source": {"$eq": "guide.md"}},
            {"page": {"$lte": 10}},
            {"langue": {"$eq": "fr"}}
        ]
    }
)

# OU logique
résultats = collection.query(
    query_texts=["erreur"],
    n_results=3,
    where={
        "$or": [
            {"section": {"$eq": "erreurs"}},
            {"section": {"$eq": "debugging"}}
        ]
    }
)

# IN — une valeur parmi une liste
résultats = collection.query(
    query_texts=["installation"],
    n_results=5,
    where={"source": {"$in": ["guide.md", "readme.txt", "setup.md"]}}
)

# NIN — exclure des valeurs
résultats = collection.query(
    query_texts=["configuration"],
    n_results=5,
    where={"source": {"$nin": ["deprecated.md", "old_guide.md"]}}
)
```

## Filtres sur le contenu — where_document

```python
# Contient un mot
résultats = collection.query(
    query_texts=["livraison"],
    n_results=3,
    where_document={"$contains": "express"}   # texte contient "express"
)

# Ne contient pas un mot
résultats = collection.query(
    query_texts=["garantie"],
    n_results=3,
    where_document={"$not_contains": "deprecated"}
)

# Combinaison where + where_document
résultats = collection.query(
    query_texts=["configuration"],
    n_results=3,
    where={"langue": "fr"},                       # métadonnées
    where_document={"$contains": "important"}     # contenu
)
```

## Contrôler ce que query() retourne

```python
# include contrôle les champs retournés
résultats = collection.query(
    query_texts=["retour"],
    n_results=3,
    include=[
        "documents",    # ← textes bruts
        "metadatas",    # ← métadonnées
        "distances",    # ← scores de distance
        "embeddings",   # ← vecteurs (coûteux en mémoire)
        "uris",         # ← URIs si des fichiers sont référencés
    ]
)

# Défaut : ["documents", "metadatas", "distances"]
# Enlever "distances" si tu n'en as pas besoin → plus léger
```

## Comprendre les distances Chroma

```python
# Avec hnsw:space = "cosine" (défaut recommandé)
distances = résultats["distances"][0]
# → [0.05, 0.23, 0.67]
# 0.0 = identique
# 1.0 = orthogonal (aucune relation)
# 2.0 = opposé

# Convertir en score de similarité (0 à 1)
def distance_to_score(distance: float) -> float:
    """Convertit une distance cosine en score de similarité [0, 1]."""
    return 1 - distance / 2

scores = [distance_to_score(d) for d in distances]
# → [0.975, 0.885, 0.665]
```

## Recherche avec seuil de pertinence

```python
def rechercher_avec_seuil(
    collection,
    query: str,
    seuil_similarité: float = 0.7,
    n_max: int = 10
) -> list[dict]:
    """Ne retourne que les résultats au-dessus du seuil de pertinence."""

    résultats = collection.query(
        query_texts=[query],
        n_results=n_max,
        include=["documents", "metadatas", "distances"]
    )

    docs_filtrés = []
    for doc, meta, dist in zip(
        résultats["documents"][0],
        résultats["metadatas"][0],
        résultats["distances"][0]
    ):
        similarité = 1 - dist / 2
        if similarité >= seuil_similarité:
            docs_filtrés.append({
                "texte": doc,
                "metadata": meta,
                "similarité": round(similarité, 3)
            })

    return docs_filtrés

résultats = rechercher_avec_seuil(collection, "retour produit", seuil_similarité=0.75)
for r in résultats:
    print(f"Similarité {r['similarité']} | {r['texte'][:80]}")
```

## Tableau des opérateurs de filtre

| Opérateur | Signification | Exemple |
|---|---|---|
| `$eq` | Égal à | `{"page": {"$eq": 3}}` |
| `$ne` | Différent de | `{"langue": {"$ne": "en"}}` |
| `$gt` | Supérieur à | `{"score": {"$gt": 0.8}}` |
| `$gte` | Supérieur ou égal | `{"page": {"$gte": 5}}` |
| `$lt` | Inférieur à | `{"page": {"$lt": 10}}` |
| `$lte` | Inférieur ou égal | `{"date": {"$lte": "2025-01-01"}}` |
| `$in` | Dans une liste | `{"source": {"$in": ["a.pdf", "b.pdf"]}}` |
| `$nin` | Pas dans la liste | `{"source": {"$nin": ["old.pdf"]}}` |
| `$and` | ET logique | `{"$and": [{...}, {...}]}` |
| `$or` | OU logique | `{"$or": [{...}, {...}]}` |
| `$contains` | Texte contient | `where_document={"$contains": "mot"}` |
| `$not_contains` | Texte ne contient pas | `where_document={"$not_contains": "mot"}` |

> [!tip] Filtrer avant de chercher = beaucoup plus rapide
> Sur un grand corpus, combine toujours `where` avec `query_texts`. Chroma applique le filtre metadata AVANT la recherche vectorielle, réduisant drastiquement l'espace de recherche.

> [!warning] Les valeurs metadata doivent être des types simples
> Chroma accepte uniquement `str`, `int`, `float` et `bool` dans les métadonnées. Pas de listes, pas de dicts imbriqués. Pour stocker une liste de tags, utilise une string séparée par des virgules et filtre avec `$contains`.
