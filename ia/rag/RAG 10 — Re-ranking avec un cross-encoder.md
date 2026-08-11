#ia #rag #reranking #intermédiaire

## Pourquoi la recherche seule reste approximative

La recherche d'un RAG — dense, lexicale ou hybride (voir [[RAG 02 — Embeddings et Vector Databases]] et [[RAG 08 — Hybrid RAG]]) — repose sur un **bi-encoder** : un modèle qui calcule séparément le vecteur de la question et le vecteur de chaque document, puis compare ces vecteurs déjà figés. Cette séparation est ce qui rend la recherche rapide (les vecteurs des documents sont calculés une fois, à l'indexation) — mais c'est aussi sa faiblesse : la question et le document ne sont jamais « lus ensemble ». Le modèle compare deux résumés indépendants, jamais l'un à la lumière de l'autre.

> [!warning] Un cas concret d'échec
> Sur la question « comment garder les données quand on supprime un conteneur ? », une recherche dense peut placer en tête un document sur la commande `prune` (hors sujet) et reléguer plus bas le document qui parle réellement de conserver les données — alors que le bon document est bien présent dans l'index. La recherche n'a pas manqué le document ; elle l'a simplement mal classé.

## Bi-encoder vs cross-encoder

Le **cross-encoder** travaille autrement : au lieu de calculer deux vecteurs séparés, il prend la paire complète — question et document concaténés — et la passe en une fois dans le modèle, pour en sortir un score de pertinence unique. Il peut ainsi juger si *ce* document précis répond à *cette* question précise, là où le bi-encoder compare deux photos prises séparément.

| | Bi-encoder (recherche) | Cross-encoder (re-ranking) |
|---|---------------------------|--------------------------------|
| Calcul | Vecteurs séparés | Paire lue ensemble |
| Vitesse | Rapide | Lent |
| Précision | Approximative | Élevée |
| Passe à l'échelle | Tout le corpus | Une présélection seulement |

Le cross-encoder est plus juste mais bien plus lent — l'appliquer à un corpus entier serait inenvisageable, puisqu'il faudrait le faire tourner sur chaque document à chaque question.

## Le schéma en deux temps

L'idée : combiner les deux modèles pour ce que chacun fait bien. Le bi-encoder présélectionne vite (une douzaine de candidats, pas seulement le top 3), puis le cross-encoder note chaque paire (question, candidat) de ce petit lot et on garde le top K du reclassement.

```python
def recherche_avec_reranking(question: str, k: int = 3) -> list[str]:
    """Recherche large, puis re-ranking : le pipeline en deux temps."""
    candidats = recherche_dense(question, n=5)  # présélection large
    return reclasser(question, candidats, k)     # reclassement fin
```

> [!tip] Le meilleur des deux modèles
> Le cross-encoder ne tourne que sur quelques candidats, jamais sur le corpus entier — sa lenteur devient négligeable, et sa précision profite entièrement à la sélection finale.

## Implémenter le re-ranking

```python
from sentence_transformers import CrossEncoder

MODELE_RERANK = "BAAI/bge-reranker-v2-m3"  # cross-encoder multilingue

def reclasser(question: str, candidats: list[str], k: int = 3) -> list[str]:
    """Seconde passe : le cross-encoder note chaque paire et reclasse."""
    paires = [(question, candidat) for candidat in candidats]
    scores = CrossEncoder(MODELE_RERANK).predict(paires)
    classement = sorted(zip(candidats, scores), key=lambda c: c[1], reverse=True)
    return [texte for texte, _ in classement[:k]]
```

Sur le cas d'échec présenté plus haut, le re-ranking rétablit le bon ordre : le document sur la commande `prune` (1er après la recherche dense) cède sa place au document qui parle réellement de la conservation des données après suppression d'un conteneur.

> [!warning] La qualité du reranker compte autant que le principe
> Tous les cross-encoders ne se valent pas — un reranker « base » plus léger peut se tromper là où un modèle solide et multilingue (`bge-reranker-v2-m3`) trouve le bon classement, en particulier sur un corpus francophone. Le re-ranking n'aide que si le modèle de re-ranking est, lui, à la hauteur.

## Quand le re-ranking vaut son coût

Le cross-encoder est un modèle à charger et exécuter — chaque question ajoute une passe d'inférence. Ce coût n'est pas toujours justifié.

> [!tip] Mesurer avant d'optimiser
> Le re-ranking se justifie quand le retrieval, **mesuré**, plafonne malgré une recherche correcte : un recall honnête, mais le bon document trop souvent en 2ᵉ ou 3ᵉ position plutôt qu'en tête. C'est le symptôme classique qu'une seconde passe peut corriger. Il ne se justifie pas par principe — si la recherche (surtout en mode hybride) place déjà le bon document en tête, ajouter un cross-encoder alourdit sans gagner. Voir [[RAG 07 — Graph RAG]] et [[RAG — Pièges classiques]] (piège « ajouter de la complexité trop tôt ») pour le même principe appliqué à d'autres techniques avancées.

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| Le re-ranking ne change rien | Recherche déjà bonne | Normal — le re-ranking n'a rien à corriger |
| Le re-ranking dégrade le classement | Cross-encoder faible ou non multilingue | Choisir `bge-reranker-v2-m3` ou équivalent |
| Latence trop forte | Cross-encoder appliqué à trop de candidats | Réduire la présélection (`n`) avant re-ranking |
| Premier lancement très long | Téléchargement du modèle | Normal, mis en cache pour les lancements suivants |
| Aucun gain mesuré | Re-ranking ajouté sans diagnostic préalable | Mesurer le recall avant de l'ajouter |

## Pour aller plus loin

Le re-ranking est l'une des techniques post-retrieval couvertes plus largement dans [[RAG 04 — Advanced RAG]] (avec la compression de contexte et les techniques pre-retrieval comme HyDE).

Sources : [Re-ranking pour le RAG — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/rag-reranking/)
