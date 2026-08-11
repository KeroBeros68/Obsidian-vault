#ia #qdrant #production #avancé

## Trois précautions avant d'exposer Qdrant réellement

Qdrant tient sa place du prototype à la production, mais l'usage réel demande quelques précautions au-delà d'un simple `docker run`.

### Persistance

Un volume monté sur `/qdrant/storage` (voir [[Qdrant 02 — Installation & lancement]]) est indispensable pour que l'index survive au cycle de vie du conteneur — sans lui, un redémarrage ou une recréation du conteneur efface tout l'index.

### Déploiement

```
Service simple      → docker compose
Haute disponibilité  → Kubernetes (Qdrant fournit ses propres manifestes)
```

### Sécurité

> [!warning] Ne jamais exposer un Qdrant nu sur un réseau ouvert
> L'API Qdrant se protège par une clé API, et le trafic doit être chiffré en TLS — un Qdrant accessible sans authentification sur un réseau non maîtrisé expose l'intégralité des collections (lecture, écriture, suppression) à quiconque atteint le port `6333`. Le même principe que pour toute base de données exposée sans protection réseau.

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| `Connection refused` sur le port 6333 | Qdrant non démarré | Lancer le conteneur `qdrant/qdrant` |
| Erreur de dimension à l'`upsert` | `size` de la collection ≠ dimension du modèle d'embedding | Recréer la collection à la bonne dimension |
| Résultats sans texte | Payload non demandé | Passer `with_payload=True` à `query_points` |
| Le filtre ne restreint rien | Clé de payload ou valeur incorrecte | Vérifier les noms et types dans `FieldCondition` |
| Index perdu au redémarrage | Données stockées dans le conteneur, sans volume | Monter un volume sur `/qdrant/storage` |

## Pour aller plus loin

Cela conclut le module Qdrant — voir [[Qdrant — Index des fiches]]. Pour resituer Qdrant dans l'ensemble du pipeline RAG (chunking, embeddings, génération sourcée), voir [[RAG — Index des fiches]].

Sources : [Qdrant — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/qdrant/)
