#ia #fine-tuning #évaluation #benchmarks #pratique

## Évaluer un modèle fine-tuné

Un modèle fine-tuné sans évaluation rigoureuse peut sembler meilleur sur les cas connus mais régresser sur les cas généraux. L'évaluation valide que l'amélioration est réelle et généralisable.

## Les 3 questions fondamentales

```
1. Le modèle a-t-il appris ce qu'on voulait lui apprendre ?
   → Évaluation sur le domaine cible

2. A-t-il conservé ses capacités générales ?
   → Évaluation sur des tâches générales (catastrophic forgetting)

3. Les améliorations justifient-elles le coût ?
   → Comparaison coût/bénéfice
```

## Métriques pendant l'entraînement

### Training loss et Validation loss

```
Training loss   : erreur sur les données d'entraînement
Validation loss : erreur sur les données non vues

Évolution idéale :
  Epoch 1 : train_loss=2.1, val_loss=2.3
  Epoch 2 : train_loss=1.8, val_loss=2.0
  Epoch 3 : train_loss=1.6, val_loss=1.8  ← bon équilibre

Signes d'overfitting :
  Epoch 1 : train_loss=2.1, val_loss=2.3
  Epoch 2 : train_loss=1.5, val_loss=2.1
  Epoch 3 : train_loss=1.1, val_loss=2.4  ← val_loss remonte → overfitting !
```

> [!warning] Surveiller la divergence train/val
> Si la training loss continue de baisser mais que la validation loss remonte, arrêter l'entraînement immédiatement (early stopping). Le modèle mémorise au lieu d'apprendre.

### Perplexité

```
Mesure de confiance du modèle sur ses propres prédictions.
Plus la perplexité est basse, plus le modèle est confiant.

Perplexité élevée → le modèle hésite, peu de cohérence
Perplexité basse  → le modèle est confiant sur ses prédictions

À surveiller en cours d'entraînement, pas en absolu.
```

## Évaluation post-entraînement

### Étape 1 — Évaluation sur le domaine cible

```python
def évaluer_domaine_cible(modèle_ft, dataset_eval):
    scores = []
    
    for exemple in dataset_eval:
        question = exemple["input"]
        réponse_attendue = exemple["expected_output"]
        
        # Générer la réponse du modèle fine-tuné
        réponse_ft = modèle_ft.generate(question)
        
        # Évaluer avec LLM-as-Judge
        score = llm_judge(
            question=question,
            réponse_générée=réponse_ft,
            réponse_attendue=réponse_attendue,
            critères="Exactitude, style, format, ton"
        )
        scores.append(score)
    
    return {
        "score_moyen": sum(scores) / len(scores),
        "score_médian": sorted(scores)[len(scores)//2],
        "score_min": min(scores),
        "score_max": max(scores),
        "taux_succès": sum(1 for s in scores if s >= 7) / len(scores)
    }
```

### Étape 2 — Test de régression (catastrophic forgetting)

```python
# Tester que le modèle n'a pas perdu ses capacités générales
tests_généraux = [
    {"input": "Traduis en anglais : 'Bonjour le monde'", "expected": "Hello world"},
    {"input": "Résous : 15 × 8 = ?", "expected": "120"},
    {"input": "Quel est le synonyme de 'heureux' ?", "expected": ["joyeux", "content", "ravi"]},
    {"input": "Résume en 1 phrase : [texte long]", "type": "résumé"},
]

score_général_avant = évaluer(modèle_base, tests_généraux)
score_général_après = évaluer(modèle_ft, tests_généraux)

dégradation = score_général_avant - score_général_après
if dégradation > 0.1:  # dégradation > 10%
    print("⚠️ Catastrophic forgetting détecté ! Revoir l'entraînement.")
```

### Étape 3 — Comparaison A/B structurée

```python
def comparaison_ab(modèle_base, modèle_ft, questions_test):
    résultats = {"ft_meilleur": 0, "base_meilleur": 0, "égalité": 0}
    
    for question in questions_test:
        réponse_base = modèle_base.generate(question)
        réponse_ft = modèle_ft.generate(question)
        
        # Comparaison par LLM-as-Judge (en aveugle)
        préférence = llm_comparer(question, réponse_base, réponse_ft)
        
        if préférence == "B":
            résultats["ft_meilleur"] += 1
        elif préférence == "A":
            résultats["base_meilleur"] += 1
        else:
            résultats["égalité"] += 1
    
    total = len(questions_test)
    print(f"FT meilleur    : {résultats['ft_meilleur']/total:.0%}")
    print(f"Base meilleur  : {résultats['base_meilleur']/total:.0%}")
    print(f"Égalité        : {résultats['égalité']/total:.0%}")
```

## Grille d'évaluation complète

```
Avant de déployer un modèle fine-tuné, valider :

  ✅ Score domaine cible > seuil défini (ex: 0.80)
  ✅ Score général modèle FT ≥ Score général modèle base - 5%
  ✅ Comparaison A/B : FT préféré dans > 60% des cas
  ✅ Aucun comportement problématique sur les cas adversariaux
  ✅ Format de sortie conforme à 100% des cas (si formatage critique)
  ✅ Latence acceptable pour le cas d'usage
  ✅ Coût par requête justifié par l'amélioration
```

## Red teaming — chercher les failles

Tester activement les cas où le modèle pourrait échouer.

```
Tests à effectuer systématiquement :
  → Questions hors domaine (doit refuser ou rediriger)
  → Requêtes dans d'autres langues
  → Questions avec des fautes d'orthographe importantes
  → Formulations ambiguës ou piégées
  → Inputs très courts ("?", "aide")
  → Inputs très longs (dépasse le contexte)
  → Tentatives d'injection de prompt
  → Demandes de comportements non souhaités
```

> [!tip] L'évaluation humaine reste irremplaçable
> Pour les tâches subjectives (ton, créativité, pertinence), une évaluation humaine sur 50-100 exemples reste le gold standard. L'automatisation complète de l'évaluation a ses limites.
