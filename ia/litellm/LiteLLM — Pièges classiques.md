#ia #litellm #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier que delta.content peut être None en streaming

```python
# ❌ Plante sur certains fragments du flux
for chunk in response:
    print(chunk.choices[0].delta.content, end="")
```

> [!warning] Toujours prévoir le cas où le fragment ne contient pas de texte
> Certains chunks du flux (métadonnées de fin, par exemple) n'ont pas de `content`. Utiliser `chunk.choices[0].delta.content or ""` pour retomber sur une chaîne vide plutôt que de tenter d'afficher `None`. Voir [[LiteLLM 02 — Streaming]].

---

## 🪤 Piège 2 — Retenter une AuthenticationError

```python
# ❌ Le backoff exponentiel s'applique à une erreur qui ne se résoudra jamais en réessayant
for attempt in range(max_retries):
    try:
        response = completion(...)
    except AuthenticationError:
        time.sleep(2 ** attempt)  # Inutile : la clé restera invalide
```

> [!warning] Une clé API invalide est une erreur de configuration, pas un incident transitoire
> Contrairement au rate limiting, réessayer une requête après une `AuthenticationError` échouera systématiquement de la même façon. Faire échouer immédiatement plutôt que de consommer du temps en tentatives inutiles. Voir [[LiteLLM 05 — Gestion des erreurs]].

---

## 🪤 Piège 3 — Laisser l'historique grossir sans limite

```python
messages = [{"role": "system", "content": SYSTEM_PROMPT}]
# ... 200 échanges plus tard, `messages` contient des centaines d'entrées
```

> [!warning] Chaque appel renvoie (et facture) tout l'historique
> Une conversation longue consomme progressivement plus de tokens à chaque tour, jusqu'à risquer de dépasser la fenêtre de contexte du modèle. Prévoir une stratégie de troncature ou de résumé de l'historique au-delà d'un certain nombre d'échanges. Voir [[LiteLLM 03 — Conversation avec historique]].

---

## 🪤 Piège 4 — Commiter la clé API dans .env sans .gitignore

```bash
# ❌ .env créé, jamais ajouté au .gitignore
echo "OPENAI_API_KEY=sk-proj-..." > .env
git add .
git commit -m "setup"
```

> [!warning] Une clé poussée sur un dépôt public est repérée en quelques minutes
> Toujours ajouter `.env` au `.gitignore` **avant** d'y écrire la clé, pas après. Voir [[LiteLLM 01 — Premier appel & anatomie de la réponse]].

---

## 🪤 Piège 5 — Function calling avec Ollama via le mauvais provider

```python
# ❌ L'agent reboucle indéfiniment sur le même outil, sans jamais répondre
completion(model="ollama/llama3.2", messages=messages, tools=SCHEMA)

# ✅ ollama_chat/ passe par /api/chat, seul à gérer le va-et-vient des messages "tool"
completion(model="ollama_chat/llama3.2", messages=messages, tools=SCHEMA)
```

> [!warning] `ollama/` et `ollama_chat/` ne sont pas interchangeables pour le function calling
> LiteLLM expose deux providers Ollama : `ollama/` est l'intégration historique, `ollama_chat/` passe par l'endpoint `/api/chat`. Seul le second gère correctement le renvoi des messages de rôle `tool` au modèle — avec `ollama/`, un agent avec des outils reboucle sur le même appel sans jamais produire de réponse finale, un symptôme qui ressemble à un bug de la boucle agent mais vient en réalité du choix du provider. Voir [[LiteLLM 06 — Function calling et outils]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `delta.content` non géré en streaming | `chunk.choices[0].delta.content or ""` |
| Retry sur une `AuthenticationError` | Échec immédiat, pas de backoff |
| Historique de conversation illimité | Prévoir troncature/résumé au-delà d'un seuil |
| Clé API committée par oubli | `.gitignore` avant d'écrire le `.env` |
| Function calling qui reboucle avec Ollama | Utiliser `ollama_chat/<modèle>`, pas `ollama/<modèle>` |
