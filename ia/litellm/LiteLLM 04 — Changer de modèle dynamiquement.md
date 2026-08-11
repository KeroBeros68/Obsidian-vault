#ia #litellm #fondamentaux

## Comparer plusieurs providers sans changer le code

```python
from litellm import completion
from dotenv import load_dotenv
load_dotenv()

QUESTION = "Donne 3 bonnes pratiques pour écrire un Dockerfile"

MODELES = [
    "gpt-4.1-mini",
    "claude-3-5-haiku-latest",
    "ollama/llama3.2",  # Nécessite Ollama installé, voir [[Ollama — Index des fiches]]
]

for modele in MODELES:
    print(f"\n{'='*50}")
    print(f"🤖 Réponse de {modele}")
    print('='*50)
    try:
        response = completion(
            model=modele,
            messages=[{"role": "user", "content": QUESTION}],
            max_tokens=300
        )
        print(response.choices[0].message.content)
        print(f"\n📊 Tokens: {response.usage.total_tokens}")
    except Exception as e:
        print(f"❌ Erreur: {e}")
```

Seul le paramètre `model` change d'une itération à l'autre — le reste de la logique applicative (construction des messages, traitement de la réponse) reste strictement identique.

> [!tip] Cas d'usage : arbitrer entre qualité et coût
> Faire tourner la même question sur plusieurs modèles permet de comparer concrètement la qualité de réponse au coût réel (voir `response.usage.total_tokens` de [[LiteLLM 01 — Premier appel & anatomie de la réponse]]) — utile avant de figer un choix de modèle pour une fonctionnalité donnée, plutôt que de se fier à des benchmarks génériques qui ne reflètent pas forcément le cas d'usage réel.

## Convention de nommage des modèles

LiteLLM identifie le fournisseur soit par le nom du modèle lui-même (`gpt-4.1-mini`, `claude-3-5-haiku-latest`), soit par un préfixe explicite pour les providers auto-hébergés (`ollama/llama3.2`). Cette convention `provider/modèle` est ce qui permet à une seule fonction `completion()` de router vers le bon backend sans configuration supplémentaire.

## Pour aller plus loin

Comparer des modèles suppose que les erreurs (clé invalide, rate limit, provider indisponible) soient gérées proprement plutôt que de faire planter le script — couvert dans [[LiteLLM 05 — Gestion des erreurs]].

Sources : [LiteLLM : une API unifiée pour tous les LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/litellm/)
