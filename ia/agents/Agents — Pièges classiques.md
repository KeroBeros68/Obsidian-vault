#ia #agents #pièges #erreurs #debugging

## 🪤 Piège 1 — Pas de guardrails → boucle infinie

```python
# ❌ Dangereux : pas de limite d'itérations
agent = AgentExecutor(agent=agent, tools=tools)

# ✅ Toujours définir des limites
agent = AgentExecutor(
    agent=agent,
    tools=tools,
    max_iterations=10,        # max 10 boucles
    max_execution_time=60,    # timeout 60 secondes
    early_stopping_method="generate"
)
```

> [!warning] Sans limite, un agent peut boucler indéfiniment
> Coût illimité en tokens, ressources saturées, et l'agent peut prendre des actions répétitives non souhaitées.

---

## 🪤 Piège 2 — System prompt trop vague

```
# ❌ Trop vague → comportement imprévisible
system_prompt = "Tu es un assistant utile."

# ✅ Précis sur le rôle, les règles et le format
system_prompt = """Tu es un assistant de support client pour Acme Corp.
Tu réponds aux questions sur nos produits.
Tu ne discutes jamais des concurrents.
Tu escalades vers un humain si : remboursement > 100€, plainte formelle, menace légale.
Format de réponse : toujours terminer par une question de satisfaction."""
```

> [!tip] Mémo
> Le system prompt d'un agent est son "contrat de travail". Plus il est précis, plus le comportement est prévisible et fiable.

---

## 🪤 Piège 3 — Trop d'outils disponibles

```
# ❌ 15 outils → le LLM choisit mal, les descriptions se confondent
tools = [recherche_web, rag, sql, api_crm, api_slack, email, 
         calendrier, fichiers, code, calcul, pdf, ...]

# ✅ Commencer avec 3-5 outils essentiels
tools = [recherche_rag, appel_api_crm, envoyer_email]
# Ajouter seulement si un besoin précis est identifié
```

> [!warning] La loi de Hick
> Plus il y a d'options, plus la décision est lente et sujette à erreur. S'applique aux LLM comme aux humains.

---

## 🪤 Piège 4 — Actions irréversibles sans validation

```python
# ❌ L'agent peut envoyer des emails en production directement
@tool
def envoyer_email(dest: str, sujet: str, corps: str):
    """Envoie un email."""
    smtp.send(dest, sujet, corps)  # action immédiate et irréversible

# ✅ Mode draft + validation humaine pour les actions critiques
@tool
def créer_brouillon_email(dest: str, sujet: str, corps: str):
    """Crée un brouillon d'email pour validation avant envoi."""
    return gmail.create_draft(dest, sujet, corps)
```

> [!warning] Principe d'irréversibilité
> Toute action que tu ne peux pas annuler (envoi d'email, suppression, paiement, post sur les réseaux) doit passer par une validation humaine.

---

## 🪤 Piège 5 — Ignorer la gestion des erreurs d'outils

```python
# ❌ Pas de gestion d'erreur → l'agent plante ou boucle
@tool
def appel_api(url: str) -> str:
    """Appelle une API externe."""
    return requests.get(url).json()

# ✅ Toujours retourner un message d'erreur clair
@tool
def appel_api(url: str) -> str:
    """Appelle une API externe."""
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except Exception as e:
        return f"Erreur lors de l'appel à {url}: {str(e)}. Essaie une approche différente."
```

> [!tip] Messages d'erreur actionnables
> Quand un outil échoue, le message d'erreur doit dire au LLM quoi faire ensuite, pas juste signaler l'échec.

---

## 🪤 Piège 6 — Context window saturée sur les longues tâches

```
# ❌ Accumuler toutes les étapes sans limite
historique = [étape_1(50k tokens), étape_2(40k tokens), étape_3(...)...]
→ Dépasse la context window → erreur ou dégradation silencieuse

# ✅ Résumer périodiquement l'historique
if len(historique) > seuil:
    résumé = llm.résumer(historique)
    historique = [résumé]  # remplacer par le résumé compact
```

---

## 🪤 Piège 7 — Pas de traçabilité en production

```
# ❌ Aucun log → impossible de déboguer ou d'auditer
agent.run(tâche)

# ✅ Logger chaque étape : prompt, outil appelé, paramètres, résultat, latence
# Outils : LangSmith, Langfuse, Arize Phoenix
```

> [!info] La traçabilité est obligatoire en production
> Sans traces, un agent qui fait une erreur est impossible à déboguer. Tu ne sais pas quelle étape a échoué, pourquoi, et comment corriger.

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Boucle infinie | max_iterations + timeout |
| System prompt vague | Rôle + règles + format précis |
| Trop d'outils | 3-5 outils max, ajouter progressivement |
| Actions irréversibles | Mode draft + human-in-the-loop |
| Erreurs d'outils non gérées | Try/catch + message d'erreur actionnable |
| Context window saturée | Résumé périodique de l'historique |
| Pas de traçabilité | LangSmith, Langfuse, ou logs custom |
