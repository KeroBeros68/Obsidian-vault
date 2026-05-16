#matplotlib #plot #bar #scatter #hist #graphiques

## Line plot
```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, y,
    color="steelblue", linewidth=2,
    linestyle="--",    # "-", "--", "-.", ":"
    marker="o",        # "o","s","^","*","D"
    markersize=4,
    label="Série A"
)
# Raccourci : "r--o" = rouge, tirets, cercles
```

## Bar plot
```python
bars = ax.bar(categories, valeurs,
    color="steelblue", edgecolor="black", width=0.6)
ax.bar_label(bars, fmt="%d", padding=3)   # valeurs sur barres
ax.barh(categories, valeurs)              # horizontal
```

## Scatter plot
```python
scatter = ax.scatter(x, y,
    s=tailles, c=couleurs,    # taille et couleur variables
    cmap="viridis", alpha=0.7,
    edgecolors="black", linewidths=0.5
)
fig.colorbar(scatter, ax=ax, label="Valeur")
```

## Histogram
```python
ax.hist(data, bins=30,
    color="steelblue", edgecolor="black",
    alpha=0.7, density=False)   # density=True → densité
ax.axvline(data.mean(), color="red",
           linewidth=2, linestyle="--")
```

## Pie chart
```python
ax.pie(valeurs, labels=labels,
    explode=[0.1,0,0,0],     # décaler une part
    autopct="%1.1f%%",       # afficher %
    startangle=90)
```

## Lignes de référence
```python
ax.axhline(y=0, color="black", linewidth=0.8)
ax.axvline(x=0, color="black", linewidth=0.8)
ax.axhspan(10, 20, alpha=0.2, color="green")
ax.axvspan(2, 4,  alpha=0.2, color="red")
```

## Récapitulatif
| Graphique | Usage |
|---|---|
| `ax.plot()` | Évolution temporelle, tendances |
| `ax.bar()` | Comparer des catégories |
| `ax.scatter()` | Relation entre 2 variables |
| `ax.hist()` | Distribution d'une variable |
| `ax.pie()` | Proportions (< 5 catégories) |
