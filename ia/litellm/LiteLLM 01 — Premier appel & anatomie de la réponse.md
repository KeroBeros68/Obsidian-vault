#ia #litellm #fondamentaux

## Mise en place

```bash
mkdir -p ~/lab-litellm && cd ~/lab-litellm
python -m venv venv && source venv/bin/activate
pip install litellm python-dotenv
```

```bash
# .env
OPENAI_API_KEY=sk-proj-votre-cle-ici
ANTHROPIC_API_KEY=sk-ant-...   # Optionnel, pour tester plusieurs providers

echo ".env" >> .gitignore
```

> [!warning] Ne jamais commiter une clé API
> Toujours passer par un fichier `.env` ajouté au `.gitignore`, ou par des variables d'environnement système — jamais une clé en dur dans le code source.

> [!tip] Alternative gratuite sans clé API payante
> Installer Ollama (voir [[Ollama — Index des fiches]]) et utiliser `model="ollama/llama3.2"` permet de suivre tous les exemples de ce module sans dépense — LiteLLM route l'appel vers l'instance locale au lieu d'une API cloud.

## Premier appel

```python
from litellm import completion
from dotenv import load_dotenv
load_dotenv()

response = completion(
    model="gpt-4.1-mini",
    messages=[
        {"role": "user", "content": "Explique la différence entre Docker et une VM en 3 lignes."}
    ]
)

print(response.choices[0].message.content)
```

## Anatomie de la réponse

```python
response.choices[0].message.content   # Le texte généré
response.model                         # Le modèle réellement utilisé, ex. "gpt-4.1-mini"
response.usage.total_tokens            # Tokens consommés (entrée + sortie)
response._hidden_params.get('response_cost', 0)  # Coût estimé de cet appel
```

> [!info] Suivre le coût par appel dès le premier script
> `_hidden_params['response_cost']` donne une estimation du coût de la requête sans avoir à consulter le tableau de bord du fournisseur — utile pour prendre l'habitude de logger cette valeur avant même de mettre une application en production (voir [[LLMOps — Index des fiches]] pour le suivi des coûts à plus grande échelle).

## Pour aller plus loin

Une réponse générée d'un bloc reste peu agréable pour un usage interactif — l'afficher progressivement, mot par mot, est couvert dans [[LiteLLM 02 — Streaming]].

Sources : [LiteLLM : une API unifiée pour tous les LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/litellm/)
