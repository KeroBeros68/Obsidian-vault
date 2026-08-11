#ia #qdrant #avancé

## Recherche sémantique avec query_points

```python
def rechercher(client, question, k=3, filtre=None):
    """Recherche les k documents les plus proches, avec filtre optionnel."""
    resultat = client.query_points(
        COLLECTION,
        query=vectoriser([question])[0],
        limit=k,
        query_filter=filtre,
        with_payload=True,
    )
    return [
        {"texte": p.payload["texte"], "sujet": p.payload["sujet"], "score": p.score}
        for p in resultat.points
    ]
```

`query_points` renvoie les points classés par similarité décroissante, chacun avec son score et son payload.

> [!warning] Sans with_payload=True, pas de texte
> `with_payload=True` est ce qui ramène le texte et les métadonnées associées à chaque point. Sans cette option, `query_points` ne renvoie que des identifiants et des scores — aucun contenu exploitable.

## Filtrer par payload : le mécanisme du multi-tenant

C'est la capacité qui distingue vraiment Qdrant des bases vectorielles embarquées (voir [[Qdrant 01 — Qu'est-ce que Qdrant]]) : le filtre restreint la recherche aux points dont le payload satisfait une condition — la recherche sémantique ne s'applique qu'à ce sous-ensemble.

```python
from qdrant_client.models import FieldCondition, Filter, MatchValue

def filtre_sujet_annee(sujet, annee):
    """Construit un filtre Qdrant : payload sujet ET année."""
    return Filter(
        must=[
            FieldCondition(key="sujet", match=MatchValue(value=sujet)),
            FieldCondition(key="annee", match=MatchValue(value=annee)),
        ]
    )

# Chercher uniquement dans les documents Docker de 2026
resultats = rechercher(client, "conteneur", k=5,
                        filtre=filtre_sujet_annee("docker", 2026))
```

Un `Filter` se compose de `FieldCondition`, chacune une condition sur une clé du payload.

| Clause | Rôle |
|--------|------|
| `must` | Toutes les conditions doivent être vraies |
| `should` | Au moins une condition doit être vraie |
| `must_not` | Aucune des conditions ne doit être vraie |

> [!info] Le filtre s'applique avant la recherche sémantique
> Un point hors filtre n'est jamais examiné par la recherche vectorielle — donc jamais renvoyé, quel que soit son score de similarité potentiel. Ce n'est pas un post-filtrage sur les résultats, mais une restriction de l'espace de recherche en amont.

## Le multi-tenant en pratique

Plusieurs équipes ou clients peuvent partager une seule collection, chaque recherche restant cloisonnée à son périmètre par un filtre systématique sur une clé (`equipe_id`, `client_id`...) — c'est la pierre angulaire de la sécurité d'un RAG en production partagé entre plusieurs utilisateurs ou organisations.

> [!warning] Ce filtre doit s'appliquer ici, jamais dans le prompt du LLM
> Récupérer tous les documents puis demander au modèle d'ignorer ceux hors périmètre n'est pas une protection : le passage interdit est déjà dans le contexte, exposé à une injection de prompt. Voir [[RAG 11 — Sécurité, évaluation & observabilité en production]] pour ce principe détaillé.

## Pour aller plus loin

Le passage de ce prototype à une instance réellement exploitée en production — persistance, déploiement, sécurité — est couvert dans [[Qdrant 05 — Qdrant en production]].

Sources : [Qdrant — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/qdrant/)
