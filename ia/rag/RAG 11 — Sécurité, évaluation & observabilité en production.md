#ia #rag #production #sécurité #avancé

## Un RAG qui marche sur un poste n'est pas un RAG en production

Trois exigences séparent un prototype d'un RAG réellement exploité : la **sécurité** (un utilisateur ne récupère que ce à quoi il a droit), l'**évaluation** (savoir si le système s'améliore ou se dégrade) et l'**observabilité** (voir ce qui se passe à chaque requête). Ce sont les mêmes piliers que pour toute application LLM en production (voir [[LLMOps — Index des fiches]]), appliqués spécifiquement au fonctionnement d'un RAG.

## Sécurité : filtrer par droits d'accès, jamais dans le prompt

Un RAG d'entreprise indexe des documents de sensibilités différentes — notes publiques, documents d'équipe, données confidentielles. Un RAG naïf cherche dans tout l'index, avec un risque direct de faire fuiter, dans une réponse, un passage que l'utilisateur n'avait pas le droit de voir.

```python
# L'utilisateur appartient à l'équipe « infra » :
# la recherche sémantique ne s'applique qu'à ses documents.
resultats = client.query_points(
    COLLECTION,
    query=vectoriser([question])[0],
    limit=k,
    query_filter=filtre_equipe("infra"),  # voir Qdrant 04
    with_payload=True,
)
```

Le mécanisme est le filtrage par métadonnée — payload sous Qdrant (voir [[Qdrant 04 — Rechercher & filtrer par payload]]), déjà couvert pour la construction du filtre lui-même. Ce qui compte ici est **où** ce filtre s'applique.

> [!warning] Le filtrage se fait à la recherche, jamais dans le prompt
> Récupérer tous les documents puis demander au modèle « ignore ceux de l'équipe X » est une faille, pas une protection : le passage interdit est déjà présent dans le contexte envoyé au modèle, et une injection de prompt (voir [[LLMOps 08 — Sécurité et guardrails en production]]) peut le faire ressortir. Le seul filtrage sûr **exclut** le document avant qu'il n'atteigne le modèle, au niveau de la requête vectorielle elle-même — un document hors filtre n'est jamais récupéré, donc jamais injecté, donc jamais dans la réponse.

C'est le principe du **multi-tenant** : plusieurs équipes ou clients partagent la même base vectorielle, mais chaque recherche reste cloisonnée à son périmètre par un filtre systématique.

## Évaluation : recall du retrieval vs exactitude de la réponse

Un RAG peut échouer à deux endroits distincts : la recherche ne remonte pas le bon document, ou la réponse déforme un document pourtant correct. Les distinguer dit où corriger.

```python
JEU_TEST = [
    {"question": "Comment conserver les données d'un conteneur ?",
     "source": "volumes.md", "terme": "volume"},
    {"question": "Comment des conteneurs communiquent par leur nom ?",
     "source": "reseau.md", "terme": "réseau"},
]

def recall_at_k(source_attendue: str, sources_trouvees: list[str]) -> bool:
    """Vrai si la source attendue figure parmi les sources récupérées."""
    return source_attendue in sources_trouvees

def contient_terme(reponse: str, terme: str) -> bool:
    """Vrai si la réponse contient le terme factuel attendu."""
    return terme.lower() in reponse.lower()

def evaluer(index, jeu_test, k=3):
    """Évalue le RAG sur le jeu de test : recall et exactitude."""
    recalls, exactitudes = [], []
    for cas in jeu_test:
        resultat = repondre(index, cas["question"], k)
        recalls.append(recall_at_k(cas["source"], resultat["sources"]))
        exactitudes.append(contient_terme(resultat["reponse"], cas["terme"]))
    n = len(jeu_test)
    return {"recall_at_k": sum(recalls) / n, "exactitude": sum(exactitudes) / n}
```

| Symptôme | Cause | Action |
|----------|-------|--------|
| Recall bas | Le retrieval échoue | Revoir le chunking (voir [[RAG 09 — Chunking en pratique (taille fixe, phrases, sections)]]), ajouter la recherche hybride (voir [[RAG 08 — Hybrid RAG]]) |
| Exactitude basse, recall correct | La génération échoue malgré un bon contexte | Durcir le prompt (voir [[RAG 03 — Naive RAG]]), changer de modèle |

> [!tip] Des métriques simples mais honnêtes
> `recall_at_k` et `contient_terme` sont déterministes, sans modèle, faciles à comprendre — largement suffisantes pour détecter une régression. Des outils dédiés vont plus loin (fidélité au contexte, pertinence jugée par un modèle — voir [[LLMOps 03 — Évaluation des LLM (Evals)]] pour le LLM-as-Judge et les evals par règles), mais commencer par des métriques simples reste préférable : on ne mesure bien que ce qu'on comprend.

## Intégrer l'évaluation à l'amélioration continue

Mesurer une fois ne sert à rien — la valeur vient de la répétition à chaque changement (nouveau chunking, autre modèle d'embedding, prompt modifié).

> [!tip] Un test d'intégration continue pour la qualité, pas seulement pour le code
> Avant de déployer une modification, relancer le jeu de test : si le recall ou l'exactitude baissent, c'est une régression, on ne déploie pas. Voir [[LLMOps 03 — Évaluation des LLM (Evals)]] pour l'intégration des evals en CI/CD de façon plus générale.

## Observabilité : journaliser chaque requête

En production, le RAG répond à des questions qu'on n'a pas anticipées. Journaliser chaque requête sous une forme structurée (pas une ligne de texte libre) permet d'agréger, filtrer et analyser après coup — voir [[LLMOps 06 — Observabilité et tracing]] pour les niveaux d'observabilité applicables à toute application LLM.

```python
def journaliser(question: str, resultat: dict) -> dict:
    """Construit un enregistrement structuré d'une requête."""
    return {
        "question": question,
        "sources": resultat["sources"],
        "latence_ms": resultat["latence_ms"],
        "longueur_reponse": len(resultat["reponse"]),
    }
```

> [!info] Trois informations qui méritent de figurer systématiquement
> Les **sources récupérées** (pour repérer un document jamais utile, ou au contraire surreprésenté), la **latence** (pour suivre les performances), et de quoi **relier la requête à un retour utilisateur** éventuel. Agrégés, ces journaux révèlent ce qu'aucun test ne montre : les questions fréquentes mal couvertes, les pics de latence, les documents morts.

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| Une réponse cite un document interdit | Filtrage absent ou fait dans le prompt | Filtrer par métadonnée à la recherche, jamais après |
| Impossible de comparer deux versions | Pas de jeu de test | Construire un jeu de questions de référence |
| Recall bas | Retrieval défaillant | Revoir le chunking, ajouter la recherche hybride |
| Exactitude basse, recall correct | Génération défaillante | Durcir le prompt, changer de modèle |
| Dérive invisible en production | Pas de journalisation | Journaliser chaque requête en structuré |

## Pour aller plus loin

Cela conclut le parcours pratique du module RAG — voir [[RAG — Index des fiches]] pour une vue d'ensemble complète.

Sources : [RAG en production — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/rag-production/)
