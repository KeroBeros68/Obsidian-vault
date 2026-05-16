#matplotlib #couleurs #styles #annotations #légende

## Couleurs
```python
ax.plot(x, y, color="steelblue")   # nom anglais
ax.plot(x, y, color="#2E86AB")     # hex
ax.plot(x, y, color=(0.2,0.4,0.8)) # RGB tuple
ax.plot(x, y, color="r")           # raccourci
ax.plot(x, y, color="steelblue", alpha=0.5)  # transparence
```

## Styles de lignes et marqueurs
```python
linestyle : "-"  "--"  "-."  ":"
marker    : "o"  "s"   "^"   "*"  "D"  "+"
# Raccourci : "r--o" = rouge + tirets + cercles
```

## Annotations
```python
ax.text(x=2, y=0.5, s="Mon texte",
    fontsize=12, color="red",
    ha="center", va="bottom", fontweight="bold")

ax.annotate("Point important",
    xy=(3.14, 0),         # cible (où pointe la flèche)
    xytext=(4, 0.5),      # position du texte
    arrowprops=dict(arrowstyle="->", color="black", lw=1.5))
```

## Légende
```python
ax.legend(loc="upper right")
ax.legend(loc="best")
ax.legend(loc="upper left", bbox_to_anchor=(1,1))  # hors graphique
ax.legend(fontsize=11, framealpha=0.9, ncol=2)
```

## Axes — personnalisation
```python
ax.set_xlim(0, 10)
ax.set_xticks([0, 5, 10])
ax.set_xticklabels(["Jan","Juin","Déc"], rotation=45)
ax.tick_params(axis="both", labelsize=11)
ax.set_xscale("log")              # échelle log
ax.spines["top"].set_visible(False)    # supprimer bordures
ax.spines["right"].set_visible(False)
ax.set_axisbelow(True)            # grille derrière les données
```

## Grille
```python
ax.grid(True, which="major", axis="y",
    color="gray", linestyle="--",
    linewidth=0.5, alpha=0.4)
```

## Styles globaux
```python
plt.style.use("seaborn-v0_8")
plt.style.use("ggplot")
plt.style.use("fivethirtyeight")

with plt.style.context("ggplot"):   # temporaire
    fig, ax = plt.subplots()
    ax.plot(x, y)
```

## Colormaps essentielles
| Type | Exemples |
|---|---|
| Séquentielle | `viridis`, `plasma`, `Blues` |
| Divergente | `coolwarm`, `RdBu` |
| Qualitative | `tab10`, `Set2`, `Paired` |
