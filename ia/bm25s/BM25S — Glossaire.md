#ia #bm25s #glossaire #référence

| Terme | Définition |
|---|---|
| **BM25** | Okapi BM25 (Best Match 25). Algorithme de ranking de documents basé sur la fréquence des termes et leur rareté dans le corpus. Standard de la recherche d'information depuis les années 90. |
| **BM25S** | Implémentation Python de BM25 utilisant des matrices sparse SciPy et un scoring "eager" pour atteindre jusqu'à 500× le speedup de rank-bm25. |
| **rank-bm25** | Librairie Python historique pour BM25. Simple mais lente sur les grands corpus. Utilisée par le BM25Retriever natif de LangChain. |
| **TF (Term Frequency)** | Fréquence d'apparition d'un terme dans un document. Plus un terme apparaît, plus le document est potentiellement pertinent — jusqu'à un certain seuil (contrôlé par k1). |
| **IDF (Inverse Document Frequency)** | Mesure de la rareté d'un terme dans le corpus. Un terme rare (ex: "bm25s") a un IDF élevé → plus de poids qu'un terme commun (ex: "le"). |
| **Eager scoring** | Stratégie de bm25s : pré-calcule et stocke les scores de tous les tokens au moment de l'indexation. La recherche devient un simple lookup → vitesse maximale. |
| **Sparse matrix** | Matrice creuse SciPy. Stocke uniquement les valeurs non-nulles, économisant mémoire et temps de calcul. Clé de la performance de bm25s. |
| **k1** | Paramètre BM25 contrôlant le poids de la fréquence des termes. Défaut : 1.5. Élevé = termes fréquents très avantagés. Faible = saturation rapide. |
| **b** | Paramètre BM25 contrôlant la normalisation par la longueur du document. Défaut : 0.75. b=1.0 = normalisation complète. b=0.0 = pas de normalisation. |
| **BM25Okapi** | Variante standard de BM25. Défaut de bm25s. Bon point de départ pour la plupart des cas. |
| **BM25L** | Variante qui pénalise moins les longs documents. Utile si ton corpus a des documents de longueur très variable. |
| **BM25Plus** | Variante qui garantit un score minimal positif pour chaque terme correspondant. Réduit le biais contre les courts documents. |
| **Tokenisation** | Découpage du texte en unités (tokens) — généralement des mots. Étape préalable à l'indexation BM25. |
| **Stemming** | Réduction des mots à leur racine commune. "retour", "retours", "retourner" → "retour". Améliore le recall. Nécessite PyStemmer. |
| **Stopwords** | Mots vides sans valeur informative ("le", "de", "est"...) à exclure avant l'indexation. BM25S inclut des listes pour plusieurs langues. |
| **PyStemmer** | Librairie Python de stemming basée sur l'algorithme Snowball. Supporte le français, anglais, allemand, espagnol et d'autres langues. |
| **recall** | En recherche d'information : proportion de documents pertinents effectivement retrouvés. Le stemming améliore le recall. |
| **precision** | Proportion de documents retournés qui sont réellement pertinents. À équilibrer avec le recall. |
| **Recherche lexicale** | Recherche basée sur la correspondance exacte de mots-clés. BM25S est une recherche lexicale. Complémentaire à la recherche sémantique (vectorielle). |
| **Recherche sémantique** | Recherche basée sur le sens, via des embeddings vectoriels. Trouve "panne de démarrage" pour "voiture ne démarre pas". |
| **Hybrid RAG** | Combinaison de BM25S (lexical) et d'un vectorstore (sémantique) via EnsembleRetriever et RRF. Standard recommandé en production. |
| **EnsembleRetriever** | Classe LangChain qui fusionne plusieurs retrievers via Reciprocal Rank Fusion. `weights` contrôle l'influence relative. |
| **RRF (Reciprocal Rank Fusion)** | Algorithme de fusion de listes de résultats. Chaque document reçoit un score basé sur sa position dans chaque liste, puis les scores sont sommés. |
| **Memory-mapped (mmap)** | Technique de lecture des fichiers qui évite de tout charger en RAM. `mmap=True` dans `BM25.load()` pour les grands index. |
| **Index persistant** | Fichiers sur disque contenant l'index bm25s (matrices sparse + paramètres). Permet de recharger sans reconstruire. |
| **numba backend** | Backend optionnel de bm25s utilisant la compilation JIT Numba. Donne environ 2× de speedup supplémentaire sur les grands corpus. |
| **BM25Retriever** | Classe LangChain native qui utilise rank-bm25 sous le capot. Moins performante que BM25SRetriever (custom) sur les grands corpus. |
| **BM25SRetriever** | Classe custom (voir fiche 05) qui implémente l'interface Runnable de LangChain avec bm25s comme moteur. |
| **camelCase** | Convention de nommage où chaque mot commence par une majuscule sauf le premier : `getUserById`. Le tokenizer code doit séparer ces mots pour indexer correctement. |
| **snake_case** | Convention de nommage où les mots sont séparés par des underscores : `error_handler`. Le tokenizer code splitte sur `_`. |
| **PascalCase** | Variante de camelCase où le premier mot commence aussi par une majuscule : `DatabaseConnection`. |
| **tokenizer_code** | Fonction de tokenisation custom qui sépare camelCase, snake_case et PascalCase pour indexer du code source correctement. |
| **RecursiveCharacterTextSplitter.from_language()** | Méthode LangChain qui crée un TextSplitter optimisé pour un langage de programmation donné (Python, JS, Go...). Coupe aux frontières naturelles du code. |
| **CodeBERT** | Modèle d'embedding Microsoft spécialisé pour le code source. Plus précis que les embeddings généralistes pour la recherche sur codebase. |
| **corpus bilingue** | Corpus contenant des documents en plusieurs langues. Stratégie recommandée : deux retrievers BM25S séparés (un par langue) fusionnés via EnsembleRetriever. |
| **all-MiniLM-L6-v2** | Modèle d'embedding sentence-transformers rapide et performant pour l'anglais. Bon compromis docs + code en anglais. |
| **"porter"** | Ancien stemmer anglais (algorithme Porter). Disponible dans PyStemmer mais remplacé par `"english"` (Snowball) plus précis. Utilise `"english"` sauf compatibilité système. |
