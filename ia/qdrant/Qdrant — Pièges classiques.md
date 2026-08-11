#ia #qdrant #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier with_payload=True

```python
resultat = client.query_points(COLLECTION, query=vecteur, limit=5)
# ❌ resultat.points[0].payload est vide : aucun texte exploitable
```

> [!warning] Sans cette option, seuls les scores reviennent
> `with_payload=True` est indispensable pour récupérer le texte et les métadonnées de chaque point trouvé — sans lui, `query_points` ne renvoie que des identifiants et des scores. Voir [[Qdrant 04 — Rechercher & filtrer par payload]].

---

## 🪤 Piège 2 — Changer de modèle d'embedding sans recréer la collection

```python
# Collection créée pour un modèle 768 dimensions (nomic-embed-text)
# ❌ Tentative d'upsert avec des vecteurs 1536 dimensions (autre modèle)
```

> [!warning] La dimension est fixée à la création de la collection
> `size` dans `VectorParams` doit correspondre exactement à la dimension du modèle d'embedding utilisé. Changer de modèle sans recréer la collection provoque une erreur de dimension à l'écriture. Voir [[Qdrant 03 — Collections & indexation avec payload]].

---

## 🪤 Piège 3 — Lancer Qdrant sans volume monté

```bash
docker run -d --name qdrant -p 6333:6333 qdrant/qdrant
# ❌ Aucun volume : l'index disparaît à la suppression du conteneur
```

> [!warning] Les données vivent dans le conteneur par défaut
> Sans `-v qdrant_storage:/qdrant/storage`, tout redémarrage ou recréation du conteneur efface l'index — un piège classique en développement qui devient critique s'il survit jusqu'en production. Voir [[Qdrant 02 — Installation & lancement]].

---

## 🪤 Piège 4 — Exposer Qdrant sans authentification

```bash
# ❌ Port 6333 accessible depuis Internet, sans clé API ni TLS
docker run -d -p 0.0.0.0:6333:6333 qdrant/qdrant
```

> [!warning] Accès total et anonyme à toutes les collections
> Sans clé API et sans TLS, quiconque atteint le port peut lire, modifier ou supprimer n'importe quelle collection. Voir [[Qdrant 05 — Qdrant en production]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `with_payload` oublié | Toujours le passer à `True` pour récupérer le contenu |
| Modèle d'embedding changé sans recréer la collection | Recréer avec la bonne dimension |
| Pas de volume monté | `-v qdrant_storage:/qdrant/storage` |
| Qdrant exposé sans protection | Clé API + TLS, jamais nu sur un réseau ouvert |
