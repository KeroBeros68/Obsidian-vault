#ia #rag #glossaire #référence

| Terme | Définition |
|---|---|
| **RAG** | Retrieval-Augmented Generation. Technique qui enrichit un LLM avec des données externes récupérées en temps réel. |
| **Retrieval** | La phase de récupération : chercher les passages pertinents dans une base de données avant de les envoyer au LLM. |
| **Chunk** | Morceau de document découpé pour l'indexation. Taille typique : 256 à 1024 tokens. |
| **Chunking** | Le processus de découpage des documents en chunks avant indexation. |
| **Chunking récursif** | Découpage en cascade sur des séparateurs (paragraphe puis phrase puis mot), respectant autant que possible la structure du texte — le choix par défaut de la plupart des frameworks. |
| **Chunking par phrases** | Découpage qui regroupe des phrases entières sans jamais en couper une, jusqu'à un budget de mots — toujours lisible, au prix d'un dépassement possible sur une phrase très longue. |
| **Chunking par sections** | Découpage qui suit la structure explicite d'un document (titres Markdown, sections HTML) — un chunk par section, la stratégie la plus qualitative quand le document s'y prête. |
| **Recouvrement (overlap)** | Répétition des derniers mots d'un chunk au début du suivant, pour qu'une idée à cheval sur la frontière entre deux chunks reste intacte dans au moins un des deux. |
| **Chunking sémantique** | Découpage basé sur la proximité de sens entre phrases plutôt que sur une taille fixe, coupant là où le sujet change réellement. |
| **FAISS** | Librairie de recherche vectorielle en mémoire développée par Meta, optimisée pour la vitesse sur de grands volumes — sans les fonctionnalités de persistance ou de filtrage d'une base vectorielle complète comme Chroma ou Qdrant. |
| **Embedding** | Représentation numérique d'un texte sous forme de vecteur (liste de nombres). Deux textes similaires ont des embeddings proches. |
| **Vector Database** | Base de données spécialisée dans le stockage et la recherche par similarité de vecteurs. Ex : Chroma, Pinecone, Qdrant. |
| **Similarité cosinus** | Mesure de proximité entre deux vecteurs, de 0 (aucun rapport) à 1 (identiques). Utilisée pour retrouver les chunks les plus pertinents. |
| **Top-K** | Le paramètre qui définit combien de chunks pertinents on récupère. Typiquement entre 3 et 10. |
| **Re-ranking** | Étape post-retrieval où un second modèle reclasse les résultats par pertinence réelle. |
| **Recall du retrieval** | Mesure si la source attendue figure parmi les chunks récupérés par la recherche — un recall bas pointe vers un problème de retrieval (chunking, absence de recherche hybride), pas de génération. |
| **Exactitude de la réponse** | Mesure si un fait attendu figure dans la réponse finale générée — distincte du recall : une exactitude basse malgré un bon recall pointe vers un problème de prompt ou de modèle, pas de recherche. |
| **Filtrage à la recherche (vs dans le prompt)** | Principe de sécurité : exclure un document interdit avant qu'il n'atteigne le modèle, au niveau de la requête vectorielle — demander au modèle d'« ignorer » un document déjà présent dans le contexte n'est pas une protection, seulement une invitation à l'injection de prompt. |
| **Bi-encoder** | Modèle calculant séparément le vecteur de la question et celui de chaque document, comparés ensuite — rapide, utilisé pour la recherche initiale, mais ne « lit » jamais la question et le document ensemble. |
| **Cross-encoder** | Modèle prenant une paire (question, document) concaténée en une seule entrée pour produire un score de pertinence unique — plus précis qu'un bi-encoder, mais trop lent pour tout un corpus, réservé au re-ranking d'une présélection. |
| **Query Rewriting** | Reformulation automatique de la question par un LLM pour améliorer les résultats de recherche. |
| **Query Expansion** | Génération de plusieurs variantes d'une question pour élargir la recherche. |
| **HyDE** | Hypothetical Document Embeddings. Génère une réponse fictive pour la question, puis cherche des documents proches de cette réponse. |
| **BM25** | Algorithme classique de recherche par mots-clés. Complémentaire à la recherche vectorielle dans le Hybrid RAG. |
| **RRF** | Reciprocal Rank Fusion. Algorithme standard pour fusionner les résultats de plusieurs méthodes de recherche. |
| **Self-correction** | Capacité d'un agent à évaluer sa propre réponse et à relancer une recherche si elle est insuffisante. |
| **Knowledge Graph** | Graphe de connaissances : réseau structuré d'entités reliées par des relations typées. Base du Graph RAG. |
| **Compression** | Suppression des parties non pertinentes dans les chunks récupérés avant de les envoyer au LLM. |
| **Context window** | La quantité maximale de texte (en tokens) qu'un LLM peut traiter en une fois. Le RAG permet de ne pas la saturer. |
| **Hallucination** | Quand un LLM invente une information. Le RAG réduit (mais n'élimine pas) ce phénomène en ancrant les réponses sur des documents réels. |
| **Grounding** | Le fait d'ancrer les réponses du LLM sur des sources documentaires précises. C'est l'objectif principal du RAG. |
| **Latence** | Temps de réponse du système. Les pipelines RAG complexes (Agentic, Modular) ont une latence plus élevée. |
| **Pipeline** | La séquence d'étapes d'un système RAG, de la question à la réponse finale. |
