#ia #llmops #prompt-management #versioning #pratique

## Prompt Management

Gérer les prompts comme du code : versioning, tests, déploiement contrôlé. Indispensable dès que plusieurs personnes travaillent sur le même système ou que les prompts changent régulièrement.

## Pourquoi gérer les prompts comme du code

```
❌ Sans prompt management :
  - Prompt stocké dans une variable Python hardcodée
  - Modification → personne ne sait ce qui a changé
  - Régression → impossible de revenir en arrière facilement
  - Collaboration → conflits, versions qui divergent
  - Déploiement → tout ou rien

✅ Avec prompt management :
  - Historique complet des modifications (qui, quoi, quand)
  - Rollback en un clic si régression
  - Tests automatiques avant chaque changement
  - Déploiement progressif (A/B test, canary)
  - Collaboration structurée
```

## Option 1 — Git (minimaliste, suffisant pour débuter)

Stocker les prompts dans des fichiers texte ou YAML versionnés avec Git.

```
prompts/
├── system_prompt_v1.txt
├── system_prompt_v2.txt     ← version en cours
├── rag_prompt.txt
├── classification_prompt.txt
└── prompts_config.yaml
```

```yaml
# prompts_config.yaml
prompts:
  support_client:
    version: "2.3.1"
    file: "system_prompt_v2.txt"
    model: "claude-sonnet-4-20250514"
    temperature: 0.3
    max_tokens: 1000
    deployed_at: "2025-03-15"
    changelog: "Ajout de la gestion des demandes de remboursement"
```

```python
# Chargement en production
import yaml

with open("prompts/prompts_config.yaml") as f:
    config = yaml.safe_load(f)

prompt_config = config["prompts"]["support_client"]
with open(f"prompts/{prompt_config['file']}") as f:
    system_prompt = f.read()
```

## Option 2 — LangSmith Hub (intermédiaire)

Plateforme dédiée de LangChain pour stocker, versionner et déployer des prompts.

```python
from langsmith import Client

client = Client()

# Pousser un prompt
client.push_prompt(
    "support-client-v2",
    object=ChatPromptTemplate.from_messages([
        ("system", "Tu es un assistant support client pour Acme Corp..."),
        ("human", "{question}")
    ])
)

# Charger en production (toujours la dernière version)
prompt = client.pull_prompt("support-client-v2")

# Charger une version spécifique (pour les rollbacks)
prompt = client.pull_prompt("support-client-v2:abc123")
```

## Option 3 — Langfuse (complet, recommandé en prod)

Plateforme LLMOps open-source avec prompt management intégré.

```python
from langfuse import Langfuse

langfuse = Langfuse()

# Créer une version de prompt
langfuse.create_prompt(
    name="support-client",
    prompt="Tu es un assistant support pour {{company}}. Réponds en {{langue}}...",
    labels=["production"],  # ou "staging", "development"
    config={
        "model": "claude-sonnet-4-20250514",
        "temperature": 0.3
    }
)

# Charger en production (label "production")
prompt = langfuse.get_prompt("support-client", label="production")
compiled = prompt.compile(company="Acme", langue="français")
```

## Template de prompt versionné

Structure recommandée pour chaque prompt géré.

```yaml
# prompt_support_client.yaml
name: support_client
version: "2.3.1"
created_by: "alice@acme.com"
created_at: "2025-03-15"
status: "production"  # draft | staging | production | deprecated

description: "Prompt principal du chatbot support client Acme"

model_config:
  model: "claude-sonnet-4-20250514"
  temperature: 0.3
  max_tokens: 800

variables:
  - nom: "company_name"
    description: "Nom de l'entreprise"
    default: "Acme Corp"
  - nom: "langue"
    description: "Langue de réponse"
    default: "français"

system_prompt: |
  Tu es un assistant support client pour {{company_name}}.
  Tu réponds toujours en {{langue}}.
  Tu ne discutes jamais de concurrents.
  Tu escalades si : remboursement > 100€, plainte formelle.

changelog:
  - version: "2.3.1"
    date: "2025-03-15"
    author: "alice"
    note: "Ajout règle remboursement"
  - version: "2.3.0"
    date: "2025-02-20"
    author: "bob"
    note: "Support multilingue"
```

## A/B test de prompts

Tester deux versions simultanément en production.

```python
import random

def get_prompt(user_id: str) -> tuple[str, str]:
    """Retourne le prompt et la version pour traçabilité."""
    
    # 20% du trafic sur la nouvelle version
    if hash(user_id) % 100 < 20:
        return prompt_v2, "v2"
    else:
        return prompt_v1, "v1"

# Dans la requête
prompt, version = get_prompt(user_id)
réponse = llm.invoke(prompt + question)

# Logger avec la version pour analyse
logger.info({
    "user_id": user_id,
    "prompt_version": version,
    "latence": ...,
    "satisfaction": ...
})
```

> [!tip] Règle d'or du prompt management
> Traite chaque modification de prompt comme un commit Git : message descriptif, tests avant merge, déploiement progressif. Un prompt en production sans historique est une dette technique.
