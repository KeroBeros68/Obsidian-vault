#seaborn #themes #styles #palette #facetgrid

## Les 5 styles
```python
sns.set_style("darkgrid")   # fond gris + grille blanche
sns.set_style("whitegrid")  # fond blanc + grille grise
sns.set_style("dark")       # fond gris, pas de grille
sns.set_style("white")      # fond blanc, pas de grille
sns.set_style("ticks")      # fond blanc + tirets axes

with sns.axes_style("whitegrid"):  # temporaire
    fig, ax = plt.subplots()
    sns.histplot(data=df, x="salaire", ax=ax)
```

## set_theme — configuration complète
```python
sns.set_theme(
    style      = "whitegrid",
    palette    = "Set2",
    font_scale = 1.2,
    rc         = {"figure.figsize": (10, 6)}
)
sns.reset_defaults()   # réinitialiser
```

## Contextes — taille selon l'usage
```python
sns.set_context("paper")     # publication — petit
sns.set_context("notebook")  # défaut — notebooks
sns.set_context("talk")      # présentation — grand
sns.set_context("poster")    # affiche — très grand
sns.set_context("talk", font_scale=1.2)
```

> [!tip] Style vs Context
> **Style** → apparence visuelle (fond, grille)
> **Context** → taille des éléments (police, lignes)

## Palettes
```python
# Qualitatives (catégories)
sns.set_palette("Set2")
sns.set_palette("tab10")
sns.set_palette("colorblind")  # ✅ accessible daltoniens

# Séquentielles (données ordonnées)
sns.set_palette("Blues")
sns.set_palette("viridis")

# Divergentes (centrées sur 0)
sns.set_palette("coolwarm")

# Personnalisée
sns.set_palette(["#2E86AB","#A23B72","#F18F01"])

# Visualiser
sns.palplot(sns.color_palette("Set2"))
```

## despine — supprimer bordures
```python
sns.despine()                        # top + right
sns.despine(left=True)               # + left
sns.despine(offset=10, trim=True)    # publication
```

## FacetGrid — grilles personnalisées
```python
g = sns.FacetGrid(df,
    col="ville", row="senior",
    hue="genre", height=4, palette="Set2")
g.map(sns.histplot, "salaire", bins=15, kde=True)
g.add_legend()
g.set_titles("{col_name} | {row_name}")
g.set_axis_labels("Salaire", "Fréquence")
```

## Recette graphique pro
```python
sns.set_theme(style="white", context="talk",
    palette="colorblind", font_scale=1.1)

fig, ax = plt.subplots(figsize=(10, 6))
# ... graphique ...
sns.despine(ax=ax, offset=5, trim=True)
ax.set_axisbelow(True)
ax.yaxis.grid(True, alpha=0.3)
plt.tight_layout()
fig.savefig("pro.pdf", bbox_inches="tight")
```
