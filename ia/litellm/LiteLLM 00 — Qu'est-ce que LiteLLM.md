#ia #litellm #fondamentaux

## Une seule syntaxe pour tous les fournisseurs de LLM

Chaque fournisseur d'IA impose sa propre bibliothèque et sa propre syntaxe : le SDK OpenAI ne s'utilise pas comme le SDK Anthropic, qui ne s'utilise pas comme l'API Google. LiteLLM (BerriAI) unifie tout cela derrière une seule fonction, `completion()`, dont la forme reprend celle de l'API OpenAI — le format le plus largement adopté comme référence de facto dans l'écosystème.

```python
from litellm import completion

# Même code pour TOUS les providers, seul le paramètre model change
response = completion(
    model="gpt-4.1-mini",  # ou "claude-3-5-haiku-latest", "ollama/llama3.2"
    messages=[{"role": "user", "content": "Comment optimiser un Dockerfile ?"}]
)
```

> [!tip] L'avantage concret : changer de modèle sans changer le code
> Tester GPT, Claude ou un modèle local via Ollama (voir [[Ollama — Index des fiches]]) revient à changer une chaîne de caractères, jamais la logique applicative — utile pour comparer qualité/coût entre providers, ou prévoir un repli (*fallback*) si un fournisseur tombe en panne.

## Où se situe LiteLLM dans l'écosystème

```
Votre code applicatif
        ↓
   completion(model=...)   ← LiteLLM traduit vers le bon format
        ↓
┌──────────┬──────────┬──────────┬─────────────┐
│  OpenAI  │  Claude  │  Gemini  │  Ollama (local) │
└──────────┴──────────┴──────────┴─────────────┘
```

LiteLLM ne remplace pas les SDK des fournisseurs : il les appelle en interne, mais expose une interface unique par-dessus — comparable au rôle joué par `SQLAlchemy` entre une application et plusieurs moteurs de bases de données (voir [[BDD — Home]]), ou par MCP entre les agents et leurs outils (voir [[MCP — Index des fiches]]).

## Format de retour uniforme

Quel que soit le provider appelé, la réponse suit la même structure :

```python
response.choices[0].message.content   # Le texte généré
response.model                        # Le modèle réellement utilisé
response.usage.total_tokens           # Tokens consommés
response._hidden_params.get('response_cost', 0)  # Coût estimé de l'appel
```

> [!info] Un format calqué sur celui d'OpenAI
> Ce format n'est pas une invention de LiteLLM — c'est celui de l'API OpenAI, devenu la référence que la plupart des outils de l'écosystème (LangChain, DSPy...) reprennent également, ce qui limite les frictions pour passer d'un outil à l'autre.

## Pour aller plus loin

Le premier appel concret, avec la mise en place d'une clé API et la lecture détaillée de la réponse, est couvert dans [[LiteLLM 01 — Premier appel & anatomie de la réponse]].

Sources : [LiteLLM : une API unifiée pour tous les LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/litellm/)
