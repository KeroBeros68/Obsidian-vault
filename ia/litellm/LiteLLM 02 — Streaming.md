#ia #litellm #fondamentaux

## Afficher la réponse au fur et à mesure

```python
from litellm import completion
from dotenv import load_dotenv
load_dotenv()

response = completion(
    model="gpt-4.1-mini",
    messages=[
        {"role": "user", "content": "Liste 5 bonnes pratiques pour un Dockerfile"}
    ],
    stream=True  # Active le streaming
)

for chunk in response:
    content = chunk.choices[0].delta.content or ""
    print(content, end="", flush=True)

print()  # Nouvelle ligne finale
```

Avec `stream=True`, la réponse n'arrive plus en un seul bloc mais par petits fragments (`chunk`) successifs — chaque fragment expose son texte via `chunk.choices[0].delta.content` plutôt que `message.content`.

> [!warning] `delta.content` peut être `None`
> Certains fragments du flux ne portent aucun texte (métadonnées de fin de flux, par exemple) — `chunk.choices[0].delta.content or ""` évite une erreur en retombant sur une chaîne vide plutôt que de tenter d'afficher `None`.

## Pourquoi le streaming compte pour l'expérience utilisateur

Sans streaming, l'utilisateur attend la génération complète avant de voir quoi que ce soit — pour une réponse longue, cette attente peut atteindre plusieurs secondes. Avec streaming, le texte apparaît progressivement, comme dans l'interface de ChatGPT ou Claude.ai, réduisant la perception d'attente même si le temps de génération total reste identique.

## Reconstituer le texte complet pendant le streaming

```python
full_response = ""
for chunk in response:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="", flush=True)
        full_response += content
```

Cette accumulation est indispensable dès qu'il faut réutiliser la réponse complète après l'affichage — par exemple pour l'ajouter à un historique de conversation (voir [[LiteLLM 03 — Conversation avec historique]]).

## Pour aller plus loin

Le streaming affiche une réponse ; pour qu'un assistant se souvienne des échanges précédents, l'historique de conversation doit être géré explicitement — couvert dans [[LiteLLM 03 — Conversation avec historique]].

Sources : [LiteLLM : une API unifiée pour tous les LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/litellm/)
