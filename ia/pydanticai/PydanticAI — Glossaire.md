#ia #pydanticai #glossaire #référence

| Terme | Définition |
|---|---|
| **PydanticAI** | Framework Python pour construire des agents à sortie typée garantie, avec injection de dépendances et retries automatiques — bâti sur Pydantic. |
| **`Agent`** | Objet central de PydanticAI, paramétré par un modèle, un type de dépendances (`deps_type`), un type de sortie (`output_type`) et des outils décorés. |
| **`output_type`** | Modèle Pydantic déclaré comme sortie attendue de l'agent — `run_sync` renvoie une instance validée de ce type, ou lève une exception. |
| **`UnexpectedModelBehavior`** | Exception levée quand le modèle n'a pas produit de sortie valide, même après épuisement du budget de retries. |
| **Tool Output** | Mode de sortie structurée par défaut : le schéma de sortie est décrit comme un outil que le modèle doit appeler — le plus compatible, recommandé pour un modèle local. |
| **Native Output** | Mode de sortie structurée s'appuyant sur le *structured output* natif du modèle — très fiable mais peu de modèles locaux le supportent. |
| **Prompted Output** | Mode de sortie structurée où le schéma est injecté dans le prompt et la réponse texte parsée — compatible avec tout modèle, le moins fiable des trois. |
| **`deps_type` / `deps`** | Mécanisme d'injection de dépendances : un type déclaré à la création de l'agent, une instance passée à chaque `run_sync`, transmise aux outils via `RunContext`. |
| **`RunContext`** | Objet typé reçu par un outil décoré `@agent.tool`, donnant accès aux dépendances injectées via `ctx.deps`. |
| **`ModelRetry`** | Exception qu'un outil ou un validateur de sortie peut lever pour renvoyer un message d'erreur au modèle, qui corrige et réessaie — dans la limite du budget `retries`. |
| **`retries`** | Budget de tentatives de l'agent, couvrant à la fois les appels d'outils (via `ModelRetry`) et la production de la sortie typée. |
| **`run_sync`** | Méthode d'exécution synchrone d'un agent PydanticAI — renvoie un résultat dont `.output` est l'instance typée attendue. |
