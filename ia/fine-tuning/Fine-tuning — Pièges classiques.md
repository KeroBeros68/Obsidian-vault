#ia #fine-tuning #pièges #erreurs #debugging

## 🪤 Piège 1 — Fine-tuner quand le prompt suffit

```
❌ Situation :
  "Le modèle ne répond pas dans le bon format → je vais fine-tuner"
  
  Résultat : 2 semaines de travail, $500, pour un résultat similaire
  à ce qu'un bon prompt avec 5 exemples aurait donné.

✅ Processus correct :
  1. Tester 10-20 variantes de prompts
  2. Ajouter 3-5 exemples en few-shot
  3. Si toujours insuffisant → envisager le fine-tuning
  
  Règle : si le few-shot marche à 80%, le fine-tuning marchera à 90%.
  Si le few-shot ne marche pas du tout, le fine-tuning ne sauvera pas non plus.
```

> [!tip] Le test du few-shot
> Mets 10 exemples parfaits dans ton prompt. Si le résultat est bon → fine-tune pour économiser des tokens. Si le résultat est mauvais → le problème vient d'ailleurs.

---

## 🪤 Piège 2 — Dataset de mauvaise qualité

```
❌ Symptômes :
  Modèle incohérent (parfois formel, parfois décontracté)
  Modèle qui mélange les langues
  Réponses incorrectes qui apparaissent souvent
  
  Cause : des exemples contradictoires ou erronés dans le dataset

✅ Avant de lancer l'entraînement :
  - Revue manuelle de 10-20% des exemples
  - Vérifier la cohérence du ton et du style
  - Valider l'exactitude factuelle des réponses
  - Supprimer les doublons et quasi-doublons
```

> [!warning] Garbage in, garbage out s'applique doublement au fine-tuning
> Un LLM amplifie ce qu'il apprend. Des erreurs dans le dataset → des erreurs systématiques dans le modèle.

---

## 🪤 Piège 3 — Overfitting par excès d'epochs

```python
# ❌ Trop d'epochs → le modèle mémorise au lieu d'apprendre
trainer = SFTTrainer(
    num_train_epochs=20,  # beaucoup trop pour 500 exemples
    ...
)

# Symptôme : training_loss très basse, val_loss remonte

# ✅ Surveiller la validation loss et utiliser l'early stopping
trainer = SFTTrainer(
    num_train_epochs=5,
    load_best_model_at_end=True,   # garde le meilleur checkpoint
    metric_for_best_model="eval_loss",
    greater_is_better=False,
    evaluation_strategy="steps",
    eval_steps=50
)
```

> [!info] Règle des epochs
> Pour 100-500 exemples : 3-5 epochs.
> Pour 500-5000 exemples : 2-3 epochs.
> Pour 5000+ exemples : 1-2 epochs.
> En cas de doute : commencer par 3.

---

## 🪤 Piège 4 — Catastrophic forgetting non détecté

```
❌ Situation :
  Fine-tuning sur des données support → modèle excellent sur le support
  Mais : le modèle ne sait plus faire de la traduction, du code, des maths
  Problème détecté 3 semaines après le déploiement via des plaintes

✅ Prévention :
  1. Tester les capacités générales AVANT le déploiement
  2. Si dégradation > 10% → réduire le learning rate
  3. Ajouter des exemples généraux dans le dataset (5-10%)
  4. Utiliser LoRA plutôt que le full fine-tuning
     (LoRA préserve mieux les capacités générales)
```

---

## 🪤 Piège 5 — Ne pas avoir de baseline claire

```
❌ Sans baseline :
  Avant : "le modèle répond bien parfois"
  Après : "le modèle répond bien souvent"
  → Impossible de quantifier l'amélioration

✅ Avec baseline :
  Avant : score moyen = 6.2/10 sur 100 questions test
  Après : score moyen = 8.1/10
  → +30% d'amélioration mesurable
  → Justifie le coût de $50 d'entraînement
```

---

## 🪤 Piège 6 — Learning rate trop élevé

```python
# ❌ Learning rate trop élevé → instabilité, perte des capacités
config = TrainingArguments(
    learning_rate=1e-3,  # trop élevé pour du fine-tuning
    ...
)

# Symptôme : training_loss oscille et ne converge pas

# ✅ Valeurs recommandées pour le fine-tuning
config = TrainingArguments(
    learning_rate=2e-4,  # pour LoRA
    # ou
    learning_rate=2e-5,  # pour full fine-tuning
    lr_scheduler_type="cosine",  # décroissance progressive
    warmup_ratio=0.03,  # montée progressive au début
    ...
)
```

---

## 🪤 Piège 7 — Fine-tuner pour mémoriser des faits

```
❌ Objectif : "Je veux que le modèle connaisse exactement notre
               catalogue de 500 produits avec leurs prix"

  Résultat : le modèle mélange les prix, invente des produits,
             et est confiant dans ses erreurs

  Explication : les LLM apprennent des patterns, pas des tables de données.
                Les faits précis ne se mémorisent pas fiablement via le FT.

✅ Solution : RAG sur le catalogue produit
  → Chaque requête récupère les vraies données → zéro hallucination sur les faits
```

---

## 🪤 Piège 8 — Ne pas inclure de cas de refus

```
❌ Dataset uniquement avec des cas où le modèle doit répondre
   → Le modèle essaie de répondre à TOUT, même les requêtes hors périmètre

✅ Inclure 10-15% d'exemples de refus appropriés :
  {"messages": [
    {"role": "system", "content": "Tu es un assistant support Acme."},
    {"role": "user", "content": "Donne-moi la recette du gâteau au chocolat"},
    {"role": "assistant", "content": "Je suis spécialisé dans le support Acme Corp. Pour cette question, je vous invite à consulter un site de cuisine. Y a-t-il quelque chose concernant vos commandes ou produits Acme que je peux vous aider ?"}
  ]}
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Fine-tuner à la place d'un bon prompt | Tester le few-shot d'abord |
| Dataset de mauvaise qualité | Revue manuelle de 10-20% |
| Trop d'epochs → overfitting | 3-5 epochs + early stopping |
| Catastrophic forgetting | Tester les capacités générales + LoRA |
| Pas de baseline | Mesurer avant et après sur dataset fixe |
| Learning rate trop élevé | 2e-4 pour LoRA, 2e-5 pour full FT |
| Fine-tuner pour mémoriser des faits | Utiliser le RAG |
| Pas de cas de refus | 10-15% d'exemples de refus dans le dataset |
