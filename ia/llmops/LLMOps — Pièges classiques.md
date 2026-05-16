#ia #llmops #pièges #erreurs #production

## 🪤 Piège 1 — Déployer sans evals

```
❌ "Ça a l'air bien lors de mes tests manuels → je déploie"
→ Comportement imprévisible sur les cas réels des utilisateurs
→ Régression impossible à détecter sans baseline

✅ Créer un dataset d'eval de 50+ cas avant le premier déploiement
   Définir un score minimum pour autoriser le déploiement
   Intégrer les evals dans le CI/CD
```

> [!warning] Un prompt sans evals est une boîte noire
> Chaque modification devient un pari. Les evals transforment les changements en décisions basées sur des données.

---

## 🪤 Piège 2 — Hardcoder les prompts dans le code

```python
# ❌ Prompt hardcodé dans le code Python
def répondre(question):
    system_prompt = "Tu es un assistant utile..."  # dans le code
    return llm.invoke(system_prompt + question)

# ✅ Prompt externalisé et versionné
def répondre(question):
    system_prompt = charger_prompt("support_client", version="production")
    return llm.invoke(system_prompt + question)
```

> [!tip] Mémo
> Un prompt en production change souvent. S'il est dans le code, chaque changement = un redéploiement. Externalisé, il peut être mis à jour en temps réel.

---

## 🪤 Piège 3 — Utiliser le même modèle pour tout

```
❌ Claude Opus pour toutes les requêtes, y compris les plus simples
→ Coûts 10-50× plus élevés que nécessaire

✅ Routage intelligent par complexité :
  Question simple (FAQ, formatage)  → Claude Haiku   ($0.25/M tokens)
  Tâche standard (chatbot, RAG)     → Claude Sonnet  ($3/M tokens)
  Analyse complexe (raisonnement)   → Claude Opus    ($15/M tokens)
```

---

## 🪤 Piège 4 — Ne pas monitorer la dérive du modèle

```
Situation réelle :
  Janvier  : score qualité = 0.87 ✅
  Février  : fournisseur met à jour le modèle silencieusement
  Mars     : score qualité = 0.71 ❌ (détecté 2 mois plus tard via plaintes)

Prévention :
  → Evals automatiques hebdomadaires en production
  → Alerte si score baisse de > 5% vs la semaine précédente
  → Pinning de version de modèle quand la stabilité est critique
```

> [!warning] Les fournisseurs LLM ne t'avertissent pas toujours
> Anthropic, OpenAI et Google mettent régulièrement à jour leurs modèles. Sans monitoring continu, une dégradation peut passer inaperçue pendant des semaines.

---

## 🪤 Piège 5 — Pas de fallback en cas de panne du LLM

```python
# ❌ Pas de gestion de la panne
def répondre(question):
    return llm_principal.invoke(question)  # si ça plante → l'app est down

# ✅ Fallback sur un modèle de secours
def répondre(question):
    try:
        return llm_principal.invoke(question)
    except APIError:
        try:
            return llm_backup.invoke(question)  # modèle de secours
        except:
            return "Notre service est temporairement indisponible. Réessayez dans quelques minutes."
```

> [!tip] Plan de continuité
> Définir à l'avance : quel modèle de secours ? Quel message si tout est down ? Quelle dégradation gracieuse est acceptable ?

---

## 🪤 Piège 6 — Ignorer les coûts jusqu'à la première facture

```
Cas réel :
  Prototype fonctionne bien → passage en production
  Volume × 100 → facture × 100
  Claude Opus + 10k requêtes/jour = ~$3000+/mois

Bonne pratique :
  → Estimer le coût à l'échelle avant de choisir le modèle
  → Définir un budget maximum et une alerte
  → Activer le prompt caching dès le départ si system prompt long
  → Audit mensuel des tokens consommés par cas d'usage
```

---

## 🪤 Piège 7 — Pas de guardrails sur les inputs

```
❌ Faire confiance aux entrées utilisateurs
→ Prompt injections, requêtes abusives, données sensibles dans les logs

✅ Toujours valider les inputs :
  - Limite de longueur
  - Détection d'injection de prompt
  - Détection de données personnelles (PII)
  - Rate limiting par utilisateur
```

---

## 🪤 Piège 8 — Évaluer sur les mêmes données qu'on a utilisées pour tuner le prompt

```
❌ Processus biaisé :
  Voir 20 exemples → écrire le prompt pour qu'il les gère bien
  Évaluer sur ces mêmes 20 exemples → score de 0.95 → "parfait !"
  En production → score réel de 0.65

✅ Séparation stricte :
  Dataset de développement (pour écrire/tuner le prompt)
  Dataset d'évaluation (indépendant, jamais vu pendant le dev)
  Dataset de régression (cas bugués à ne jamais reproduire)
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Pas d'evals | Dataset + score minimum + CI/CD |
| Prompts hardcodés | Externaliser + versionner |
| Modèle unique pour tout | Routage par complexité |
| Dérive non détectée | Evals hebdomadaires + alertes |
| Pas de fallback | Modèle backup + message dégradé |
| Coûts ignorés | Estimation + budget + alerte |
| Inputs non validés | Guardrails input systématiques |
| Eval biaisée | Séparation dev/eval/régression |
