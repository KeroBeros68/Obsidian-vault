#ia #rag #pièges #erreurs #debugging

## 🪤 Piège 1 — Chunks trop petits ou trop grands

```
❌ Chunks de 64 tokens → réponses sans contexte, LLM ne comprend pas le sens complet
❌ Chunks de 4096 tokens → trop de bruit, le passage pertinent est noyé
✅ Commencer avec 512 tokens + overlap de 50 tokens entre chunks
```

> [!warning] L'overlap est important
> Sans chevauchement entre chunks, une information à cheval sur deux chunks sera coupée et perdue. Toujours ajouter un overlap de 10-20%.

---

## 🪤 Piège 2 — Top-K trop faible ou trop élevé

```
❌ Top-K = 1 → on rate des passages complémentaires
❌ Top-K = 20 → trop d'informations, LLM confus ou context window dépassée
✅ Commencer avec Top-K = 3 à 5, ajuster selon les résultats
```

> [!tip] Mémo
> Plus les questions sont complexes et multidimensionnelles, plus le Top-K peut être élevé. Pour des questions simples et directes, un Top-K bas suffit.

---

## 🪤 Piège 3 — Mauvaise qualité des documents sources

```
❌ Indexer des PDFs scannés sans OCR → le texte est illisible pour l'embedding
❌ Indexer des documents mal structurés, avec tableaux ou colonnes complexes
✅ Pré-traiter les documents : nettoyer, normaliser, extraire le texte proprement
```

> [!warning] Garbage in, garbage out
> La qualité du RAG dépend directement de la qualité des documents indexés. Un mauvais prétraitement dégrade toute la chaîne.

---

## 🪤 Piège 4 — Ne pas tester la qualité du retrieval

```
❌ Tester seulement la réponse finale du LLM
✅ Tester d'abord si les bons chunks sont récupérés (étape retrieval seule)
```

Si les mauvais chunks sont récupérés, ce n'est pas un problème de LLM — c'est un problème de retrieval. Les corriger séparément.

> [!tip] Méthode de diagnostic
> Pose une question, inspecte les chunks récupérés AVANT de regarder la réponse. Si les chunks sont bons mais la réponse mauvaise → problème LLM/prompt. Si les chunks sont mauvais → problème retrieval.

---

## 🪤 Piège 5 — Oublier de mettre à jour l'index

```
❌ Les documents sources changent mais l'index vectoriel n'est pas mis à jour
→ Le RAG répond avec des informations obsolètes
✅ Mettre en place une pipeline de mise à jour automatique de l'index
```

> [!warning] L'index n'est pas magique
> Contrairement à une base de données classique, ajouter un document ne suffit pas — il faut le re-chunker, re-embedder et l'indexer.

---

## 🪤 Piège 6 — Ajouter de la complexité trop tôt

```
❌ Commencer directement avec un Agentic RAG ou un Graph RAG
✅ Commencer avec le Naive RAG → identifier les problèmes réels → ajouter des solutions ciblées
```

> [!tip] Mémo
> Chaque couche de complexité doit résoudre un problème identifié. Si le Naive RAG donne déjà de bons résultats, ne pas "améliorer" sans raison.

---

## 🪤 Piège 7 — Ne pas gérer les questions hors périmètre

```
❌ Laisser le LLM répondre de mémoire quand aucun chunk pertinent n'est trouvé
→ Risque d'hallucinations présentées comme des réponses documentées
✅ Détecter quand le score de similarité est trop bas et répondre :
   "Je n'ai pas trouvé d'information sur ce sujet dans la documentation."
```

> [!warning] Score de similarité minimal
> Toujours définir un seuil minimum de similarité en dessous duquel on considère qu'aucun résultat pertinent n'a été trouvé.

---

## 🪤 Piège 8 — Mélanger des vecteurs de deux modèles d'embedding différents

```
❌ Corpus indexé avec all-MiniLM, puis quelques documents ajoutés avec nomic-embed-text
→ Erreur de dimension à l'écriture, ou pire, scores de similarité incohérents sans erreur visible
```

> [!warning] Les espaces vectoriels de deux modèles ne sont pas comparables
> La question et les documents doivent être vectorisés par exactement le même modèle, à l'indexation comme à la recherche. Changer de modèle d'embedding — même pour une version « améliorée » du même modèle — impose de réindexer l'intégralité du corpus, pas seulement les nouveaux documents. Voir [[RAG 02 — Embeddings et Vector Databases]].

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Chunks mal calibrés | 512 tokens + overlap 10-20% pour commencer |
| Top-K inadapté | 3-5 par défaut, ajuster selon les cas |
| Documents mal prétraités | Nettoyer et normaliser avant indexation |
| Pas de test du retrieval | Inspecter les chunks avant la réponse finale |
| Index non mis à jour | Pipeline de mise à jour automatique |
| Complexité prématurée | Naive RAG d'abord, complexifier seulement si nécessaire |
| Questions hors périmètre | Seuil de similarité + réponse "je ne sais pas" |
| Vecteurs de deux modèles d'embedding mélangés | Réindexer tout le corpus après tout changement de modèle |
