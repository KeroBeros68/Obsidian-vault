#ia #litellm #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **LiteLLM** | Bibliothèque Python (BerriAI) exposant une fonction unique, `completion()`, pour appeler plus de 100 fournisseurs de LLM avec une syntaxe identique. |
| **`completion()`** | Fonction centrale de LiteLLM, dont la signature et le format de retour reprennent ceux de l'API OpenAI. |
| **Convention `provider/modèle`** | Format de nommage (ex. `ollama/llama3.2`) permettant à LiteLLM de router un appel vers le bon backend sans configuration additionnelle. |
| **`stream=True`** | Paramètre activant la réception de la réponse par fragments (`chunk`) plutôt qu'en un seul bloc. |
| **`chunk.choices[0].delta.content`** | Fragment de texte reçu pendant un appel en streaming — équivalent de `message.content` pour un appel non streamé. |
| **`response.usage.total_tokens`** | Nombre total de tokens consommés (entrée + sortie) par un appel, exposé de façon uniforme quel que soit le provider. |
| **`response_cost`** | Estimation du coût d'un appel, accessible via `response._hidden_params.get('response_cost')`. |
| **Backoff exponentiel** | Stratégie de nouvelle tentative où le délai d'attente double à chaque échec (`2 ** attempt`) — évite de marteler un provider déjà en rate limiting. |
| **`AuthenticationError` / `RateLimitError` / `APIConnectionError`** | Exceptions normalisées par LiteLLM, communes à tous les providers, à la place des exceptions propres à chaque SDK. |
| **Function calling** | Protocole par lequel un LLM renvoie un appel structuré (nom d'outil + arguments JSON) plutôt que de l'exécuter lui-même — c'est le code appelant qui valide puis exécute. |
| **`tools=`** | Paramètre de `completion()` listant le schéma JSON des outils disponibles, généré depuis un modèle Pydantic via `model_json_schema()`. |
| **`message.tool_calls`** | Liste des appels d'outils demandés par le modèle dans sa réponse — normalisée par LiteLLM quel que soit le provider. |
| **`tool_call_id`** | Identifiant reliant un message de rôle `tool` (le résultat) à l'appel précis qui l'a déclenché — indispensable dès que le modèle demande plusieurs outils en un seul tour. |
| **`ollama_chat/`** | Provider LiteLLM pour Ollama qui passe par l'endpoint `/api/chat` — seul à gérer correctement le function calling, contrairement au provider `ollama/` historique. |
