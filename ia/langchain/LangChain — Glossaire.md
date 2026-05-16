#ia #langchain #glossaire #référence

| Terme | Définition |
|---|---|
| **LangChain** | Framework Python/JS open-source pour construire des applications LLM. Fournit des abstractions standardisées pour prompts, LLM, parsers, agents, RAG. |
| **LCEL** | LangChain Expression Language. Système de composition utilisant l'opérateur `\|` pour chaîner des briques Runnable. |
| **Runnable** | Interface de base de LangChain. Toute brique qui implémente `.invoke()`, `.stream()`, `.batch()` est un Runnable chaînable avec `\|`. |
| **ChatPromptTemplate** | Template de prompt pour les LLM de type chat. Définit des messages avec rôles (system, human, assistant) et variables. |
| **MessagesPlaceholder** | Composant d'un ChatPromptTemplate qui injecte dynamiquement une liste de messages (historique de conversation). |
| **StrOutputParser** | Parser qui transforme un AIMessage en string Python simple. Le plus courant. |
| **PydanticOutputParser** | Parser qui transforme la sortie LLM en objet Pydantic typé et validé. |
| **with_structured_output()** | Méthode du LLM qui force une sortie conforme à un schéma Pydantic via function calling. Plus fiable que PydanticOutputParser. |
| **Chain** | Séquence de briques Runnable connectées avec `\|`. La sortie de chaque brique devient l'entrée de la suivante. |
| **RunnablePassthrough** | Brique qui passe l'entrée intacte en sortie. Utile pour préserver des données dans une chain parallèle. |
| **RunnableParallel** | Exécute plusieurs branches en parallèle et combine leurs sorties en un dict. Créé implicitement par `{"clé1": chain1, "clé2": chain2}`. |
| **RunnableBranch** | Choisit dynamiquement quelle branch exécuter selon une condition sur l'état. |
| **RunnableLambda** | Wrapper qui transforme une fonction Python quelconque en Runnable chaînable. |
| **InMemoryChatMessageHistory** | Stockage en mémoire de l'historique d'une conversation. Perdu au redémarrage. |
| **RunnableWithMessageHistory** | Enveloppe une chain pour lui ajouter la gestion automatique de l'historique de conversation. |
| **Document** | Objet LangChain représentant un morceau de texte : `page_content` (texte) + `metadata` (source, page...). |
| **TextSplitter** | Découpe des documents en chunks adaptés à l'indexation vectorielle. |
| **chunk_size** | Taille maximale d'un chunk en caractères. Paramètre clé du TextSplitter. |
| **chunk_overlap** | Chevauchement en caractères entre deux chunks consécutifs. Préserve le contexte aux frontières. |
| **Embeddings** | Modèle qui transforme du texte en vecteur numérique. Permet la recherche sémantique. |
| **Vectorstore** | Base de données spécialisée dans le stockage et la recherche de vecteurs par similarité. |
| **Retriever** | Interface standardisée pour chercher des documents pertinents. S'obtient via `vectorstore.as_retriever()`. |
| **EnsembleRetriever** | Fusionne plusieurs retrievers (ex: BM25 + vectoriel) via Reciprocal Rank Fusion. |
| **BM25Retriever** | Retriever basé sur l'algorithme de recherche par mots-clés BM25. Complémentaire au vectoriel. |
| **MultiQueryRetriever** | Génère plusieurs reformulations de la question pour améliorer le recall. |
| **ContextualCompressionRetriever** | Compresse les chunks récupérés pour n'en garder que les parties pertinentes. |
| **@tool** | Décorateur qui transforme une fonction Python en outil utilisable par un agent LangChain. |
| **ToolNode** | Nœud LangGraph qui exécute automatiquement les outils demandés par le LLM. |
| **StateGraph** | Classe LangGraph pour construire un graphe d'états. Chaque nœud est une fonction qui modifie l'état. |
| **AgentState** | TypedDict qui définit la structure de l'état partagé entre les nœuds d'un graphe LangGraph. |
| **create_react_agent** | Factory LangGraph qui construit un agent ReAct standard (LLM → outils → LLM...). |
| **MemorySaver** | Checkpointer LangGraph qui sauvegarde l'état en mémoire entre les nœuds. |
| **thread_id** | Identifiant de session pour les graphes LangGraph avec checkpointer. Même thread_id = même fil de conversation. |
| **LangSmith** | Plateforme d'observabilité pour LangChain. Enregistre traces, métriques et évaluations. |
| **@traceable** | Décorateur LangSmith pour tracer manuellement une fonction Python dans les traces. |
| **LangServe** | Librairie qui expose n'importe quelle chain LangChain en API REST avec streaming intégré. |
| **RemoteRunnable** | Client LangServe pour consommer une API LangServe comme une chain locale. |
| **bind_tools()** | Méthode du LLM qui lui indique quels outils sont disponibles pour le function calling. |
| **partial()** | Méthode des prompt templates pour pré-remplir certaines variables à l'avance. |
| **with_fallbacks()** | Méthode du LLM pour définir des modèles de secours en cas d'erreur. |
| **with_retry()** | Méthode d'un Runnable pour réessayer automatiquement en cas d'erreur. |
