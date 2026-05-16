#seaborn #scatterplot #lineplot #regplot #pairplot #heatmap

## scatterplot — 5 dimensions
```python
sns.scatterplot(data=df, x="age", y="salaire",
    hue="ville",       # couleur
    size="experience", # taille
    style="senior",    # forme du marqueur
    palette="Set2", alpha=0.8, ax=ax)
```

## lineplot — avec intervalles de confiance
```python
sns.lineplot(data=df, x="mois", y="ventes",
    hue="region", style="region",
    markers=True, dashes=False,
    errorbar="sd", ax=ax)
# Calcule automatiquement moyenne + IC si plusieurs obs/point
```

## regplot — régression linéaire
```python
sns.regplot(data=df, x="age", y="salaire",
    scatter_kws={"alpha":0.5},
    line_kws={"color":"red"},
    ci=95,       # IC à 95%
    order=2,     # polynomiale degré 2
    lowess=True, # régression locale
    ax=ax)
```

## pairplot — toutes les relations
```python
g = sns.pairplot(df,
    hue="ville",
    vars=["age","salaire","score"],
    diag_kind="kde",
    plot_kws={"alpha":0.5})
g.fig.suptitle("Relations", y=1.02)
```

## heatmap — matrice de corrélation
```python
corr = df.corr()
mask = np.triu(np.ones_like(corr, dtype=bool))  # triangle sup

sns.heatmap(corr, mask=mask,
    annot=True, fmt=".2f",
    cmap="coolwarm", vmin=-1, vmax=1,
    center=0, square=True,
    linewidths=0.5, ax=ax)
```

## jointplot — distribution marginale
```python
g = sns.jointplot(data=df, x="age", y="salaire",
    kind="scatter",   # "kde","hex","reg","resid"
    hue="ville", height=7)
```

## relplot — figure-level
```python
g = sns.relplot(data=df, x="age", y="salaire",
    hue="ville", col="senior", row="genre",
    kind="scatter", height=4)
```
