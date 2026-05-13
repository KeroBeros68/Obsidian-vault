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
```

> [!tip] La règle des 80/20
> Le Naive RAG résout 80% des besoins. N'ajoute de la complexité que si tu identifies un problème précis qu'il ne résout pas.

> [!warning] Piège classique
> Le Naive RAG échoue souvent sur les questions mal formulées. Si tes utilisateurs ne savent pas exactement quoi demander, passe à l'Advanced RAG avec query rewriting.
