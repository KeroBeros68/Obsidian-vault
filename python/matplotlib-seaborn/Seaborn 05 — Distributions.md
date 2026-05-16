#seaborn #distributions #histplot #boxplot #violin

## Import et philosophie
```python
import seaborn as sns

# Toujours passer ax= pour intégrer dans une figure Matplotlib
fig, ax = plt.subplots(figsize=(8, 5))
sns.histplot(data=df, x="age", ax=ax)
plt.show()
```

## histplot — histogramme moderne
```python
sns.histplot(data=df, x="salaire",
    bins=20, kde=True,        # kde=True → courbe densité
    hue="ville",              # couleur par catégorie
    color="steelblue", alpha=0.7, ax=ax)
```

## kdeplot — courbe de densité
```python
sns.kdeplot(data=df, x="salaire",
    fill=True, hue="ville", alpha=0.3, ax=ax)

# Densité 2D
sns.kdeplot(data=df, x="age", y="salaire",
    fill=True, cmap="Blues", ax=ax)
```

## boxplot
```python
sns.boxplot(data=df, x="ville", y="salaire",
    hue="senior", palette="Set2",
    width=0.6, ax=ax)
```

Anatomie du boxplot :
```
─────┤  IQR  ├─────
min  Q1 med  Q3  max
          ○ ← outlier (>1.5×IQR)
```

## violinplot
```python
sns.violinplot(data=df, x="ville", y="salaire",
    hue="senior", split=True,   # moitiés si hue binaire
    inner="box", palette="Set2", ax=ax)
```

## stripplot / swarmplot
```python
sns.stripplot(data=df, x="ville", y="salaire",
    alpha=0.7, dodge=True, ax=ax)    # jitter aléatoire
sns.swarmplot(data=df, x="ville", y="salaire",
    dodge=True, ax=ax)               # sans chevauchement
```

## Superposer boxplot + stripplot
```python
sns.boxplot(data=df, x="ville", y="salaire",
    palette="Set2", width=0.5, ax=ax)
sns.stripplot(data=df, x="ville", y="salaire",
    color="black", alpha=0.5, size=4, ax=ax)
```

## displot — figure-level
```python
sns.displot(data=df, x="salaire",
    col="ville", kind="kde", height=4)
```

> [!tip] Axes-level vs Figure-level
> **Axes-level** (`histplot`, `boxplot`) → `ax=` → intégration Matplotlib
> **Figure-level** (`displot`, `catplot`) → crée sa propre figure, supporte `col=` et `row=`
