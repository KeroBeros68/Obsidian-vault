#seaborn #barplot #countplot #catplot #heatmap

## barplot — moyennes + IC
```python
sns.barplot(data=df, x="ville", y="salaire",
    hue="senior", palette="Set2",
    errorbar="sd", capsize=0.1, ax=ax)
# Calcule automatiquement la moyenne par groupe !
```

> [!tip] barplot vs bar
> `ax.bar()` → valeurs brutes
> `sns.barplot()` → moyenne + intervalle de confiance

## countplot — compter les occurrences
```python
ordre = df["ville"].value_counts().index
sns.countplot(data=df, x="ville",
    hue="senior", palette="Set2",
    order=ordre, ax=ax)

for container in ax.containers:
    ax.bar_label(container, padding=3)
```

## catplot — figure-level
```python
g = sns.catplot(data=df, x="ville", y="salaire",
    hue="senior", col="genre",
    kind="box",   # "bar","box","violin","strip","swarm","point"
    palette="Set2", height=5)
g.set_titles("{col_name}")
```

## pointplot — moyennes connectées
```python
sns.pointplot(data=df, x="mois", y="ventes",
    hue="region", dodge=True,
    markers=["o","s","^"],
    linestyles=["-","--","-."],
    errorbar="sd", ax=ax)
```

## heatmap sur pivot table
```python
pivot = df.pivot_table(values="ventes",
    index="mois", columns="region", aggfunc="mean")

sns.heatmap(pivot,
    annot=True, fmt=".0f",
    cmap="YlOrRd",
    linewidths=0.5, linecolor="white",
    cbar_kws={"label": "Ventes"}, ax=ax)
```

## clustermap — heatmap avec clustering
```python
g = sns.clustermap(corr,
    annot=True, fmt=".2f",
    cmap="coolwarm", vmin=-1, vmax=1,
    figsize=(8,8))
# Réorganise par similarité + affiche dendrogramme
```

## Trier les catégories
```python
ordre = df.groupby("ville")["salaire"].mean()\
          .sort_values(ascending=False).index
sns.barplot(data=df, x="ville", y="salaire",
    order=ordre, ax=ax)
```

## Récapitulatif fonctions catégorielles
| Fonction | Usage |
|---|---|
| `barplot` | Moyennes + IC |
| `countplot` | Compter les occurrences |
| `boxplot` | Distribution (quartiles) |
| `violinplot` | Distribution (forme) |
| `stripplot` | Tous les points avec jitter |
| `swarmplot` | Tous les points sans chevauchement |
| `pointplot` | Moyennes connectées |
| `catplot` | Figure-level, col= et row= |
| `heatmap` | Matrice de valeurs |
| `clustermap` | Heatmap + clustering |
