#ia #litellm #fondamentaux

## Un LLM n'a pas de mémoire entre deux appels

Chaque appel à `completion()` est indépendant : sans action explicite, le modèle ne « se souvient » pas de l'échange précédent. Une vraie conversation exige de renvoyer l'intégralité de l'historique à chaque nouvel appel.

```python
from litellm import completion
from dotenv import load_dotenv
load_dotenv()

SYSTEM_PROMPT = """Tu es un assistant expert en DevOps.
Réponds de manière concise et pratique."""

def chat(messages: list, user_input: str) -> str:
    """Envoie un message et retourne la réponse."""
    messages.append({"role": "user", "content": user_input})

    response = completion(
        model="gpt-4.1-mini",
        messages=messages,
        stream=True
    )

    full_response = ""
    for chunk in response:
        content = chunk.choices[0].delta.content
        if content:
            print(content, end="", flush=True)
            full_response += content
    print("\n")

    messages.append({"role": "assistant", "content": full_response})
    return full_response
```

## La liste `messages` comme mémoire de la conversation

```python
messages = [{"role": "system", "content": SYSTEM_PROMPT}]

chat(messages, "Comment sécuriser un secret dans GitLab CI ?")
chat(messages, "Et comment y accéder dans le pipeline ?")
# La deuxième question fait sens grâce au contexte de la première,
# conservé dans `messages`
```

Chaque tour ajoute deux entrées à `messages` : le message `user` envoyé, puis le message `assistant` reçu — la liste grandit à chaque échange et constitue l'unique mémoire de la conversation, entièrement gérée côté application, pas côté modèle.

> [!warning] L'historique complet est renvoyé à chaque appel
> Plus la conversation s'allonge, plus chaque nouvel appel transmet (et facture) l'intégralité des échanges précédents — une conversation longue consomme progressivement plus de tokens par tour, jusqu'à potentiellement dépasser la fenêtre de contexte du modèle (voir [[TR 01 — L'architecture Transformer & le mécanisme d'attention]] pour la raison technique de cette limite).

## Le rôle system : la personnalité de l'assistant

`{"role": "system", "content": ...}` cadre le comportement de l'assistant pour toute la conversation — c'est ici que se définissent la spécialité, le ton, et les instructions de sécurité (voir [[LLMOps 08 — Sécurité et guardrails en production]] pour des exemples de guardrails placés dans ce même rôle).

## Pour aller plus loin

LiteLLM devient particulièrement utile pour comparer plusieurs modèles sur la même question sans changer de code applicatif — voir [[LiteLLM 04 — Changer de modèle dynamiquement]].

Sources : [LiteLLM : une API unifiée pour tous les LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/litellm/)
