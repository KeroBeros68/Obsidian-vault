#ia #agents #patterns #architecture #intermédiaire

## Les patterns d'agents

Un pattern est une architecture de référence qui résout un type de problème donné. Connaître ces patterns évite de réinventer la roue.

## Pattern 1 — ReAct (le plus courant)

**Re**asoning + **Act**ing. L'agent alterne réflexion et action.

```
Thought  : "Je dois connaître le cours actuel de l'euro"
Action   : recherche_web("cours euro dollar aujourd'hui")
Observation: "1 EUR = 1,09 USD"
Thought  : "J'ai l'info. Je peux calculer la conversion."
Action   : calculatrice("1500 * 1.09")
Observation: "1635"
Thought  : "J'ai le résultat final."
Réponse  : "1 500 € correspondent à 1 635 USD."
```

**Quand utiliser** : tâches simples à modérées nécessitant des outils. Standard par défaut.

---

## Pattern 2 — Plan and Execute

L'agent **planifie d'abord entièrement**, puis exécute.

```
Phase 1 — Planification :
  LLM génère un plan complet :
    Étape 1 : Récupérer les ventes du mois
    Étape 2 : Calculer la croissance vs mois précédent
    Étape 3 : Identifier le top 3 des produits
    Étape 4 : Rédiger le résumé
    Étape 5 : Envoyer au manager

Phase 2 — Exécution :
  Exécution séquentielle de chaque étape
  Chaque étape peut être confiée à un sous-agent spécialisé
```

**Quand utiliser** : tâches longues et complexes où la planification upfront est bénéfique.

---

## Pattern 3 — Reflexion (auto-critique)

L'agent génère une réponse, puis un second LLM (ou lui-même) la **critique et l'améliore**.

```
[Agent]  → Produit une réponse v1
    ↓
[Critique] → "La partie 2 est incomplète, les sources manquent"
    ↓
[Agent]  → Améliore → Réponse v2
    ↓
[Critique] → "Satisfaisant"
    ↓
Réponse finale
```

**Quand utiliser** : tâches de rédaction, génération de code, analyse — où la qualité prime sur la vitesse.

---

## Pattern 4 — LATS (Language Agent Tree Search)

L'agent explore **plusieurs chemins en parallèle** et choisit le meilleur.

```
Objectif
    ↓
Chemin A ──→ Résultat A (score: 7/10)
Chemin B ──→ Résultat B (score: 9/10) ← choisi
Chemin C ──→ Résultat C (score: 5/10)
```

**Quand utiliser** : problèmes d'optimisation, recherche de la meilleure solution parmi plusieurs approches. Coûteux en tokens.

> [!info] LATS vs Tree-of-Thought (ToT)
> Les deux explorent plusieurs branches de raisonnement en parallèle avant de choisir la meilleure — la différence est que ToT reste une technique de *prompting pur* (le modèle génère et note plusieurs approches en un ou quelques appels, sans agir sur un environnement réel), alors que LATS combine cette exploration arborescente avec de vraies actions d'agent et leurs retours (outils, observations) à chaque nœud de l'arbre. En pratique, un ToT simplifié tient souvent dans un seul prompt qui demande au modèle de générer 3 approches, les noter, puis trancher — utile pour un problème de décision qui ne nécessite pas d'outils externes.

---

## Pattern 5 — Human in the Loop

L'agent s'arrête à certaines étapes pour **demander validation** à un humain.

```
Agent travaille...
    ↓
[Point de contrôle] → "Je vais envoyer cet email. Voulez-vous valider ?"
    ↓
Humain valide ou corrige
    ↓
Agent continue
```

**Quand utiliser** : actions irréversibles (envoi d'emails, paiements, suppressions), décisions à fort impact, déploiement en production.

---

## Pattern 6 — Subagents (décomposition)

Un agent principal délègue des sous-tâches à des **agents spécialisés**.

```
[Agent principal]
    ↓ délègue
    ├── [Agent Recherche] → collecte les informations
    ├── [Agent Analyse]   → traite les données
    └── [Agent Rédaction] → rédige le rapport final
    ↓ reçoit les résultats et synthétise
[Rapport final]
```

**Quand utiliser** : tâches parallélisables, bénéfice à avoir des agents ultra-spécialisés. Voir aussi [[Agents 06 — Multi-agents]].

---

## Choisir le bon pattern

| Situation | Pattern recommandé |
|---|---|
| Tâche simple avec outils | ReAct |
| Tâche longue et structurée | Plan and Execute |
| Qualité de sortie critique | Reflexion |
| Actions irréversibles | Human in the Loop |
| Tâches parallèles spécialisées | Subagents |
| Optimisation / meilleure solution | LATS |

> [!tip] Les patterns se combinent
> Un agent Plan and Execute peut utiliser Human in the Loop pour les étapes critiques, et Reflexion pour améliorer la sortie finale. Les patterns sont des briques, pas des choix exclusifs.
