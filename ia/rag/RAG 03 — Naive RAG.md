#ia #rag #naive #bases

## Naive RAG

Le RAG dans sa forme la plus simple. Point de départ de tout système RAG, avant d'ajouter de la complexité.

## Pipeline complet

```
[Documents] → Chunking → Embeddings → Vector DB
                                            ↑ (indexation, une fois)

[Question] → Embedding → Recherche similarité → Top-K chunks
                                                        ↓
                                            Injection dans le prompt
                                                        ↓
                                               [LLM] → Réponse
```

## Étapes détaillées

### 1. Indexation (préparation)

```python
# Pseudo-code
documents = charger_documents("mon_dossier/")
chunks = decouper(documents, taille=512)
vecteurs = modele_embedding.encoder(chunks)
vector_db.stocker(chunks, vecteurs)
```

### 2. Requête (en temps réel)

```python
# Pseudo-code
question = "Comment retourner un produit ?"
vecteur_question = modele_embedding.encoder(question)
chunks_pertinents = vector_db.chercher(vecteur_question, top_k=3)
prompt = f"Contexte : {chunks_pertinents}\n\nQuestion : {question}"
reponse = llm.generer(prompt)
```

## Le prompt de génération : ne pas se contenter d'injecter le contexte

La ligne `prompt = f"Contexte : ..."` ci-dessus fonctionne, mais un prompt de production ajoute des instructions explicites — sans elles, rien n'empêche le LLM de répondre à partir de ses propres connaissances plutôt que du contexte fourni, ou d'halluciner une réponse quand le contexte ne contient pas l'information :

```python
prompt = f"""Tu es un assistant expert en {domaine}.
Utilise UNIQUEMENT les informations du contexte ci-dessous.
Si la réponse ne s'y trouve pas, dis-le clairement plutôt que d'inventer.

CONTEXTE :
{chunks_pertinents}

QUESTION :
{question}

INSTRUCTIONS :
- Réponds de manière concise
- Cite les parties du contexte qui soutiennent ta réponse
- N'utilise pas tes connaissances préexistantes
"""
```

> [!warning] "Injecter le contexte" ne suffit pas à garantir l'ancrage
> Sans l'instruction explicite de se limiter au contexte fourni (et de dire "je ne sais pas" si l'information est absente), le LLM peut mélanger le contexte récupéré avec ses connaissances d'entraînement — perdant l'un des principaux bénéfices du RAG : la traçabilité de la réponse à une source vérifiable.

> [!warning] Le RAG réduit les hallucinations, il ne les supprime pas
> Même cadré par un prompt strict, un modèle peut déformer le contexte fourni ou répondre légèrement à côté — le RAG diminue fortement le risque d'hallucination, il ne l'annule pas. C'est précisément pour cela que **citer les sources** (voir ci-dessous) reste essentiel : la réponse doit rester vérifiable par l'utilisateur, pas seulement plausible.

## Tracer les sources jusqu'à la réponse

Pour qu'une réponse soit vérifiable, chaque chunk indexé doit porter sa source d'origine (nom de fichier, titre de section), renvoyée avec la réponse finale — un mécanisme direct à implémenter avec le payload d'une base vectorielle comme Qdrant (voir [[Qdrant 03 — Collections & indexation avec payload]]) :

```python
sources = sorted({chunk["source"] for chunk in chunks_pertinents})
```

> [!tip] Vectoriser en un seul lot, pas chunk par chunk
> Vectoriser tous les chunks d'un document en un seul appel (`vectoriser(liste_de_chunks)`) plutôt qu'un par un est nettement plus rapide — un appel groupé au modèle d'embedding amortit son overhead sur l'ensemble du lot. Ce réflexe vaut pour toute indexation, quelle que soit la base vectorielle utilisée.

## Tester un pipeline RAG

Un RAG se teste à deux niveaux, qui ne demandent pas les mêmes outils :

- **Fonctions déterministes** — le découpage (voir [[RAG 09 — Chunking en pratique (taille fixe, phrases, sections)]]), la construction du prompt : se vérifient sans aucun modèle. Un test unitaire classique suffit à vérifier qu'un prompt contient bien le contexte et la question, par exemple.
- **Intégration** — indexer un petit corpus connu, poser une question dont la réponse est connue à l'avance, vérifier que la réponse générée contient l'information attendue et cite la bonne source.

> [!tip] La logique pure se teste vite et reste stable
> Séparer ces deux niveaux évite de faire dépendre chaque test d'un appel réel à un LLM (lent, parfois non déterministe) — la majorité de la logique (chunking, construction de prompt, filtrage) se valide par des tests rapides et stables, réservant les tests d'intégration, plus lourds, à la validation de bout en bout.

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Simple à comprendre et implémenter | Qualité dépend directement de la formulation |
| Bon point de départ | Pas de vérification de la pertinence |
| Fonctionne pour 80% des cas simples | Peut récupérer des chunks redondants |
| Peu de dépendances | Aucune gestion des questions complexes |

## Quand l'utiliser

- ✅ POC (proof of concept) et prototypage
- ✅ Base de connaissances simple et homogène
- ✅ Questions directes avec réponses localisées dans les documents
- ❌ Questions complexes nécessitant plusieurs sources
- ❌ Documents très hétérogènes en qualité ou format

## Outils pour démarrer

```
LlamaIndex + Chroma + OpenAI embeddings
→ Un RAG fonctionnel en ~50 lignes de Python

Pour un pipeline 100% local et prêt à durer :
qdrant-client + Ollama (nomic-embed-text + qwen2.5)
→ Voir le module [[Qdrant — Index des fiches]]
```

> [!tip] La règle des 80/20
> Le Naive RAG résout 80% des besoins. N'ajoute de la complexité que si tu identifies un problème précis qu'il ne résout pas.

> [!warning] Piège classique
> Le Naive RAG échoue souvent sur les questions mal formulées. Si tes utilisateurs ne savent pas exactement quoi demander, passe à l'Advanced RAG avec query rewriting.
