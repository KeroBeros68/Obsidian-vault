#matplotlib #seaborn #ml #confusion-matrix #roc #learning-curves

> [!warning] Prérequis : scikit-learn
> Cette fiche utilise `sklearn.metrics` et `sklearn.model_selection`.

## Matrice de confusion
```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_true, y_pred)

fig, ax = plt.subplots(figsize=(6,5))
sns.heatmap(cm,
    annot=True, fmt="d", cmap="Blues",
    xticklabels=["Non","Oui"],
    yticklabels=["Non","Oui"], ax=ax)
ax.set_xlabel("Prédit")
ax.set_ylabel("Réel")
ax.set_title("Matrice de confusion")
```

Lecture :
```
              Prédit Non  Prédit Oui
Réel Non  →  [  TN ✅  |   FP ❌  ]
Réel Oui  →  [  FN ❌  |   TP ✅  ]
Diagonale = prédictions correctes
```

## Courbe ROC
```python
from sklearn.metrics import roc_curve, roc_auc_score

fpr, tpr, _ = roc_curve(y_true, y_scores)
auc = roc_auc_score(y_true, y_scores)

ax.plot(fpr, tpr, color="steelblue", lw=2,
    label=f"ROC (AUC={auc:.3f})")
ax.plot([0,1],[0,1],"--", color="gray",
    label="Aléatoire (AUC=0.5)")
ax.fill_between(fpr, tpr, alpha=0.1, color="steelblue")
```

Interprétation AUC :
```
1.0  → parfait
>0.9 → excellent
>0.7 → correct
0.5  → aléatoire — inutile !
<0.5 → inverser les prédictions
```

## Learning curves
```python
from sklearn.model_selection import learning_curve

train_sizes, train_scores, val_scores = learning_curve(
    model, X, y, cv=5,
    train_sizes=np.linspace(0.1, 1.0, 10))

ax.plot(train_sizes, train_scores.mean(axis=1), label="Train")
ax.fill_between(train_sizes,
    train_scores.mean(1) - train_scores.std(1),
    train_scores.mean(1) + train_scores.std(1), alpha=0.15)
ax.plot(train_sizes, val_scores.mean(axis=1), label="Validation")
```

Diagnostic :
```
Grand écart train/val → OVERFITTING (modèle trop complexe)
Scores faibles partout → UNDERFITTING (modèle trop simple)
```

## Importance des features
```python
importances = model.feature_importances_
indices = np.argsort(importances)[::-1]

sns.barplot(x=importances[indices],
            y=features[indices],
            palette="Blues_r", ax=ax)
```

## Réel vs Prédit
```python
ax.scatter(y_test, y_pred, alpha=0.5)
lims = [min(y_test.min(), y_pred.min()),
        max(y_test.max(), y_pred.max())]
ax.plot(lims, lims, "r--", label="Prédiction parfaite")
# Points sur la ligne = prédictions parfaites
```

## Résidus
```python
residus = y_test - y_pred

ax.scatter(y_pred, residus, alpha=0.5)
ax.axhline(0, color="red", linewidth=1.5, linestyle="--")
# Résidus centrés sur 0 = bon modèle
```
