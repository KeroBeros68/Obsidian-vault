#matplotlib #export #publication #savefig #rcparams

## savefig — options essentielles
```python
fig.savefig("graph.png",
    dpi         = 300,          # résolution
    bbox_inches = "tight",      # ← TOUJOURS ajouter !
    facecolor   = "white",      # fond blanc
    transparent = False,
    pad_inches  = 0.1
)
```

> [!warning] Toujours bbox_inches="tight"
> Sans ça, les labels et titres peuvent être coupés !

## Formats selon l'usage
| Usage | Format | DPI |
|---|---|---|
| Web / écran | PNG | 72-150 |
| Présentation | PNG | 150-200 |
| Impression | PNG | 300 |
| Publication | PDF / SVG | Vectoriel |
| Édition post | SVG | Vectoriel |
| Rapport multi-pages | PdfPages | 300 |

> [!warning] Jamais JPEG pour des graphiques
> Compression avec perte → artefacts sur textes et lignes !

## rcParams — configuration globale
```python
plt.rcParams.update({
    "figure.figsize"    : (10, 6),
    "figure.dpi"        : 150,
    "font.size"         : 12,
    "axes.titlesize"    : 14,
    "axes.spines.top"   : False,
    "axes.spines.right" : False,
    "axes.grid"         : True,
    "grid.alpha"        : 0.3,
})
plt.rcdefaults()   # réinitialiser
```

## PdfPages — rapport multi-pages
```python
from matplotlib.backends.backend_pdf import PdfPages

with PdfPages("rapport.pdf") as pdf:
    for col in colonnes:
        fig, ax = plt.subplots(figsize=(10,6))
        sns.histplot(data=df, x=col, ax=ax)
        ax.set_title(col)
        pdf.savefig(fig, bbox_inches="tight")
        plt.close(fig)   # ← libère la mémoire !
```

## Plotly — graphiques interactifs
```python
import plotly.express as px

fig = px.scatter(df, x="age", y="salaire",
    color="ville", hover_data=["nom"])
fig.show()
fig.write_html("dashboard.html")
fig.write_image("graph.png", scale=2)
```

> [!tip] Matplotlib vs Plotly
> **Matplotlib/Seaborn** → statique, publication, contrôle total
> **Plotly** → interactif, web, dashboards

## Checklist publication-ready
- [ ] `sns.set_theme(style="white", context="paper")`
- [ ] `palette="colorblind"` pour l'accessibilité
- [ ] Titres et labels avec unités
- [ ] `sns.despine(offset=5, trim=True)`
- [ ] `ax.set_axisbelow(True)` + grille légère
- [ ] `plt.tight_layout()`
- [ ] `fig.savefig("fig.pdf", bbox_inches="tight")`
- [ ] `plt.close(fig)` pour libérer la mémoire
