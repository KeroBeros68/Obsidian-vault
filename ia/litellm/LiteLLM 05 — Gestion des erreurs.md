#ia #litellm #avancé

## Les exceptions normalisées de LiteLLM

Autre bénéfice de l'API unifiée : les erreurs remontées par les différents providers sont normalisées sous des exceptions communes, plutôt que de devoir gérer les erreurs propres à chaque SDK.

```python
from litellm import completion
from litellm.exceptions import (
    AuthenticationError,
    RateLimitError,
    APIConnectionError
)
from dotenv import load_dotenv
import time
load_dotenv()

def ask_with_retry(question: str, max_retries: int = 3) -> str:
    """Pose une question avec gestion des erreurs et retries."""
    for attempt in range(max_retries):
        try:
            response = completion(
                model="gpt-4.1-mini",
                messages=[{"role": "user", "content": question}],
                timeout=30
            )
            return response.choices[0].message.content

        except AuthenticationError:
            raise Exception("❌ Clé API invalide. Vérifiez OPENAI_API_KEY.")

        except RateLimitError:
            wait = 2 ** attempt  # Backoff exponentiel
            print(f"⏳ Rate limit atteint, attente {wait}s...")
            time.sleep(wait)

        except APIConnectionError:
            print(f"🔌 Connexion échouée, tentative {attempt + 1}/{max_retries}")
            time.sleep(1)

    raise Exception("❌ Échec après plusieurs tentatives")
```

## Trois catégories d'erreurs, trois traitements différents

| Exception | Cause | Traitement approprié |
|-----------|-------|------------------------|
| `AuthenticationError` | Clé API invalide, absente ou expirée | Échec immédiat — retenter ne change rien, l'erreur est de configuration |
| `RateLimitError` | Trop de requêtes envoyées au provider | Backoff exponentiel (`2 ** attempt`) : attendre de plus en plus longtemps entre chaque tentative |
| `APIConnectionError` | Provider injoignable (panne, réseau) | Nouvelle tentative après une courte pause, avec un nombre de tentatives limité |

> [!warning] Ne jamais retenter une AuthenticationError
> Contrairement au rate limiting ou aux problèmes de connexion réseau, une clé API invalide ne se résout pas en réessayant — chaque tentative supplémentaire échouera de la même façon. Faire échouer immédiatement et signaler l'erreur de configuration plutôt que de consommer du temps en retries inutiles.

> [!info] `timeout=30` protège contre un appel qui ne répond jamais
> Sans timeout explicite, une requête bloquée côté provider peut immobiliser l'application indéfiniment. Fixer une limite de temps raisonnable transforme un blocage silencieux en erreur explicite (`APIConnectionError` ou équivalent), que le code peut alors gérer.

## Pour aller plus loin

Ce module couvre le socle du fil rouge « DevOps Buddy » (appel simple, streaming, historique, multi-modèle, erreurs). La suite de la série — appels asynchrones, embeddings pour la recherche documentaire, et un routeur de production avec fallbacks et caching — fait l'objet de guides distincts, non encore couverts dans ce vault, voir [[Manques]].

Sources : [LiteLLM : une API unifiée pour tous les LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/litellm/)
