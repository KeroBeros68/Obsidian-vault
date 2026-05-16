#ia #bm25s #pièges #erreurs #debugging

## 🪤 Piège 1 — Tokenizer différent entre indexation et requête

```python
# ❌ Stemmer à l'indexation, pas à la requête → scores incohérents
import bm25s, Stemmer

stemmer = Stemmer.Stemmer("french")
corpus_tokenisé = bm25s.tokenize(corpus, stemmer=stemmer)   # avec stemmer
retriever.index(corpus_tokenisé)

query_tokenisée = bm25s.tokenize("retour produit")          # SANS stemmer !
résultats, scores = retriever.retrieve(query_tokenisée, k=3)
# → Scores erronés car les tokens ne correspondent plus

# ✅ Même tokenizer partout
corpus_tokenisé = bm25s.tokenize(corpus, stemmer=stemmer, stopwords="french")
query_tokenisée = bm25s.tokenize("retour produit", stemmer=stemmer, stopwords="french")
```

> [!warning] Règle absolue
> La tokenisation à l'indexation et à la requête doit être **identique** : mêmes stopwords, même stemmer, même tokenizer. Une différence = scores incohérents = mauvais résultats silencieux.

---

## 🪤 Piège 2 — Oublier de passer corpus à retrieve()

```python
# ❌ Sans corpus → retourne des indices numpy, pas des textes
résultats, scores = retriever.retrieve(query_tokenisée, k=3)
print(résultats[0, 0])   # → 2  (indice, pas le texte)
print(type(résultats[0, 0]))  # → np.int64

# ✅ Passer le corpus pour avoir directement les textes
résultats, scores = retriever.retrieve(query_tokenisée, corpus=corpus, k=3)
print(résultats[0, 0])   # → "La politique de retour est de 30 jours."

# Ou utiliser les indices pour accéder au corpus manuellement
idx = int(résultats[0, 0])   # convertir en int Python
texte = corpus[idx]
```

---

## 🪤 Piège 3 — Demander plus de résultats que de documents

```python
# ❌ k > nombre de documents dans le corpus → erreur
retriever.retrieve(query_tokenisée, k=10)   # si corpus a 5 docs → erreur

# ✅ Limiter k au nombre de documents disponibles
n_docs = len(corpus)
résultats, scores = retriever.retrieve(query_tokenisée, k=min(k, n_docs))
```

---

## 🪤 Piège 4 — Tenter d'ajouter des documents à un index existant

```python
# ❌ BM25S ne supporte pas l'ajout incrémental
retriever.index(nouveaux_chunks)   # ne FUSIONNE pas, écrase l'index !

# ✅ Reconstruire l'index entièrement avec tous les documents
tous_les_chunks = anciens_chunks + nouveaux_chunks
corpus_tokenisé = bm25s.tokenize([d.page_content for d in tous_les_chunks], stemmer=stemmer)
retriever = bm25s.BM25()
retriever.index(corpus_tokenisé)
retriever.save("./index_bm25s", corpus=[d.page_content for d in tous_les_chunks])
```

> [!warning] Pas d'ajout incrémental
> Contrairement à un vectorstore (où tu peux appeler `add_documents()`), BM25S nécessite une reconstruction complète à chaque mise à jour du corpus. Planifie des reconstructions nocturnes si le corpus évolue fréquemment.

---

## 🪤 Piège 5 — Ignorer la déduplication dans le Hybrid RAG

```python
# ❌ Sans déduplication → le même chunk peut apparaître deux fois
def formater_docs_sans_dédup(docs):
    return "\n\n---\n\n".join([doc.page_content for doc in docs])
# BM25S ET vectorstore peuvent tous deux retourner le même chunk
# → Le LLM voit le même texte en double → contexte inutilement chargé

# ✅ Dédupliquer par contenu avant de formater
def formater_docs(docs):
    vus = set()
    résultat = []
    for doc in docs:
        clé = doc.page_content[:100]   # empreinte sur les 100 premiers chars
        if clé not in vus:
            vus.add(clé)
            résultat.append(f"[{doc.metadata.get('source','?')}]\n{doc.page_content}")
    return "\n\n---\n\n".join(résultat)
```

---

## 🪤 Piège 6 — Utiliser bm25s sans stemming sur du français

```python
# ❌ Sans stemming sur du texte français
corpus_tokenisé = bm25s.tokenize(corpus)   # tokenisation basique
# "retour", "retours", "retourner" → 3 tokens différents
# Requête "retourner" ne trouvera PAS le doc avec "retour"

# ✅ Toujours utiliser le stemmer pour les langues flexionnelles
import Stemmer
stemmer = Stemmer.Stemmer("french")
corpus_tokenisé = bm25s.tokenize(corpus, stemmer=stemmer, stopwords="french")
# "retour", "retours", "retourner" → tous réduits à "retour"
```

---

## 🪤 Piège 7 — Ne pas sauvegarder l'index en production

```python
# ❌ Reconstruire l'index à chaque démarrage du serveur
def démarrer_app():
    # Reconstruction sur 10k documents = 30 secondes de cold start !
    corpus_tokenisé = bm25s.tokenize(charger_documents())
    retriever = bm25s.BM25()
    retriever.index(corpus_tokenisé)

# ✅ Sauvegarder et recharger
def démarrer_app():
    if os.path.exists("./index_bm25s"):
        retriever = bm25s.BM25.load("./index_bm25s", load_corpus=True)
    else:
        corpus = charger_documents()
        corpus_tokenisé = bm25s.tokenize(corpus, stemmer=stemmer)
        retriever = bm25s.BM25()
        retriever.index(corpus_tokenisé)
        retriever.save("./index_bm25s", corpus=corpus)
```

---

## 🪤 Piège 8 — Poids EnsembleRetriever mal calibrés

```python
# ❌ Poids par défaut sans réflexion
hybrid = EnsembleRetriever(
    retrievers=[bm25s_ret, vectoriel_ret],
    weights=[0.5, 0.5]   # équilibre arbitraire
)

# ✅ Ajuster selon le type de contenu
# Documentation technique (codes, termes exacts)
weights = [0.6, 0.4]   # BM25S dominant

# FAQ / support client (questions vagues)
weights = [0.3, 0.7]   # vectoriel dominant

# Contrats / textes juridiques
weights = [0.5, 0.5]   # équilibre

# Méthode empirique : tester avec 20 questions représentatives
# et choisir le poids qui donne le meilleur score de pertinence
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Tokenizer différent index/requête | Même stemmer + stopwords partout |
| Oublier corpus dans retrieve() | Toujours passer `corpus=corpus` |
| k > nombre de documents | `k=min(k, len(corpus))` |
| Ajout incrémental | Reconstruire l'index entièrement |
| Pas de déduplication Hybrid | Filtrer par empreinte de contenu |
| Pas de stemming en français | `Stemmer.Stemmer("french")` systématiquement |
| Pas de persistance | Sauvegarder avec `retriever.save()` |
| Poids EnsembleRetriever arbitraires | Calibrer selon le type de contenu |

---

## 🪤 Piège 9 — Stemmer anglais sur du code source

```python
# ❌ Stemmer anglais + code → identifiants déformés
import Stemmer
stemmer_en = Stemmer.Stemmer("english")

corpus_tokenisé = bm25s.tokenize(
    corpus_code,
    stemmer=stemmer_en   # ← détruit les identifiants !
)
# "database"   → "databas"    ← coupé
# "configure"  → "configur"   ← coupé
# "error"      → "error"      ← ok
# "connection" → "connect"    ← coupé
# Résultat : recherche "database" ne trouve plus "database" !

# ✅ Pas de stemmer sur le code pur
corpus_tokenisé = bm25s.tokenize(
    corpus_code,
    tokenizer=tokenizer_code   # ← tokenizer custom seulement
    # Pas de stemmer, pas de stopwords
)
```

> [!warning] Règle absolue pour le code
> Ne jamais appliquer un stemmer sur du code source. Les identifiants (`getUserById`, `DatabaseConnection`) sont des termes exacts — le stemming les détruit. Utilise uniquement un tokenizer custom qui sépare camelCase et snake_case.

---

## 🪤 Piège 10 — Même embedding model pour docs et code

```python
# ❌ Embedding généraliste multilingue sur du code anglais
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)
# → Comprend mal la sémantique du code (identifiants, patterns, types)

# ✅ Embedding spécialisé selon le contenu
# Pour docs anglaises
embeddings_docs = HuggingFaceEmbeddings(model_name="sentence-transformers/all-MiniLM-L6-v2")

# Pour code source
embeddings_code = HuggingFaceEmbeddings(model_name="microsoft/codebert-base")

# Pour corpus mixte FR/EN
embeddings_multi = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)
```

---

## 🪤 Piège 11 — chunk_size trop petit pour le code

```python
# ❌ Chunk de 512 chars sur du code → coupe les fonctions en plein milieu
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=50)
# Une fonction Python de 30 lignes → 3 chunks incohérents

# ✅ Utiliser le splitter spécialisé par langage
from langchain.text_splitter import Language, RecursiveCharacterTextSplitter

splitter_python = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=1000,    # plus grand pour garder les fonctions entières
    chunk_overlap=100
)
# Coupe après les classes et fonctions complètes, pas en plein milieu
```

---

## Récapitulatif mis à jour

| Piège | Solution |
|---|---|
| Tokenizer différent index/requête | Même stemmer + stopwords partout |
| Oublier corpus dans retrieve() | Toujours passer `corpus=corpus` |
| k > nombre de documents | `k=min(k, len(corpus))` |
| Ajout incrémental | Reconstruire l'index entièrement |
| Pas de déduplication Hybrid | Filtrer par empreinte de contenu |
| Pas de stemming en français | `Stemmer.Stemmer("french")` |
| Pas de persistance | Sauvegarder avec `retriever.save()` |
| Poids EnsembleRetriever arbitraires | Calibrer selon le type de contenu |
| Stemmer sur du code source | Tokenizer custom uniquement, pas de stemmer |
| Mauvais embedding pour le code | `codebert-base` pour code, `all-MiniLM` pour docs |
| chunk_size trop petit pour le code | `from_language()` + chunk_size=1000 |
