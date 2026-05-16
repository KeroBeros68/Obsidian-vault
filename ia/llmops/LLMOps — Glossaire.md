#ia #llmops #glossaire #référence

| Terme | Définition |
|---|---|
| **LLMOps** | Large Language Model Operations. Pratiques et outils pour développer, déployer, monitorer et maintenir des apps LLM en production. |
| **Eval (évaluation)** | Test automatisé mesurant la qualité des réponses d'une application LLM. Équivalent des tests unitaires pour le LLMOps. |
| **Dataset d'évaluation** | Ensemble de paires (entrée, sortie attendue) utilisé pour mesurer la qualité de l'application de manière reproductible. |
| **LLM-as-Judge** | Technique utilisant un LLM pour évaluer la qualité des réponses d'un autre LLM selon des critères définis. |
| **Faithfulness** | Métrique RAG mesurant si la réponse générée est fidèle aux documents sources (pas d'hallucination). |
| **Answer Relevance** | Métrique mesurant si la réponse répond vraiment à la question posée. |
| **Hallucination** | Phénomène où le LLM invente des informations avec confiance. L'un des principaux risques en production. |
| **Prompt Management** | Pratique de versioning, test et déploiement contrôlé des prompts, comme du code source. |
| **Tracing** | Enregistrement de toutes les étapes d'exécution d'un appel LLM ou d'un pipeline (prompts, outils, latences, tokens). |
| **Span** | Une étape individuelle dans une trace (ex : "appel LLM", "recherche vectorielle", "re-ranking"). |
| **Observabilité** | Capacité à comprendre l'état interne d'un système à partir de ses sorties. Inclut logs, traces et métriques. |
| **Guardrail** | Mécanisme de protection qui filtre les inputs ou outputs d'une app LLM pour prévenir les comportements non souhaités. |
| **Prompt injection** | Attaque où un utilisateur malveillant tente de manipuler le LLM en injectant des instructions dans son input. |
| **Jailbreak** | Technique pour contourner les instructions de sécurité d'un LLM via des formulations créatives. |
| **Data leakage** | Fuite d'informations confidentielles (system prompt, données d'autres utilisateurs) via les réponses du LLM. |
| **Rate limiting** | Limitation du nombre de requêtes par utilisateur dans une fenêtre de temps, pour prévenir les abus. |
| **Latence p95** | Le temps de réponse en dessous duquel se trouvent 95% des requêtes. Indicateur du "vrai" ressenti utilisateur. |
| **Token budget** | Limite maximale de tokens allouée à une session ou requête. Contrôle les coûts et évite les abus. |
| **Prompt caching** | Mise en cache des tokens du system prompt côté fournisseur LLM (natif Anthropic) pour réduire les coûts de 50-90%. |
| **Cache sémantique** | Cache qui retrouve des réponses précédentes pour des questions similaires sémantiquement (pas seulement identiques). |
| **Canary deployment** | Stratégie de déploiement envoyant un petit pourcentage du trafic vers la nouvelle version avant de basculer entièrement. |
| **A/B test (prompt)** | Comparaison de deux versions de prompt en production sur des portions égales de trafic pour choisir la meilleure. |
| **Dérive du modèle** | Dégradation progressive des performances due aux mises à jour silencieuses des modèles par les fournisseurs. |
| **Online eval** | Évaluation automatique de la qualité des réponses en temps réel en production (contrairement aux evals offline pré-déploiement). |
| **LiteLLM** | Proxy Python unifiant l'interface de tous les LLM (Claude, GPT, Gemini...) pour faciliter les changements de fournisseur. |
| **Langfuse** | Plateforme LLMOps open-source couvrant traces, evals et prompt management. |
| **Ragas** | Framework d'évaluation spécialisé pour les pipelines RAG (Retrieval-Augmented Generation Evaluation). |
| **OTEL / OpenTelemetry** | Standard ouvert de traçabilité distribuée, compatible avec les principaux outils LLMOps. |
