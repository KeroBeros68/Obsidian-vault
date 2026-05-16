#matplotlib #seaborn #pièges #erreurs #debugging

## 🪤 Piège 1 — savefig après show → image vide
```python
plt.show()              # ❌ vide le buffer !
fig.savefig("g.png")    # sauvegarde une image vide

fig.savefig("g.png", bbox_inches="tight")  # ✅ AVANT show
plt.show()
```

## 🪤 Piège 2 — Oublier bbox_inches="tight"
```python
fig.savefig("g.png")                          # ❌ labels coupés
fig.savefig("g.png", bbox_inches="tight")     # ✅ tout inclus
```

## 🪤 Piège 3 — plt.title() sur le mauvais Axes
```python
fig, axes = plt.subplots(1, 2)
plt.title("Titre")         # ❌ agit sur le dernier Axes actif
axes[0].set_title("Titre") # ✅ explicite et sûr
```

## 🪤 Piège 4 — Itérer sur axes sans .flat
```python
fig, axes = plt.subplots(2, 3)
for ax in axes:          # ❌ itère sur les LIGNES (arrays)
    ax.plot(x, y)        # TypeError !

for ax in axes.flat:     # ✅ itère sur les Axes individuels
    ax.plot(x, y)
```

## 🪤 Piège 5 — plt.style.use() permanent
```python
plt.style.use("ggplot")   # ❌ modifie TOUS les graphiques suivants

with plt.style.context("ggplot"):  # ✅ temporaire
    fig, ax = plt.subplots()
    ax.plot(x, y)
```

## 🪤 Piège 6 — Oublier ax= dans Seaborn
```python
fig, ax = plt.subplots()
sns.histplot(data=df, x="age")        # ❌ crée une nouvelle figure !
sns.histplot(data=df, x="age", ax=ax) # ✅ s'intègre dans fig
```

## 🪤 Piège 7 — JPEG pour des graphiques
```python
fig.savefig("graph.jpg")   # ❌ artefacts sur textes et lignes
fig.savefig("graph.png")   # ✅ sans perte
fig.savefig("graph.pdf")   # ✅ vectoriel pour publication
```

## 🪤 Piège 8 — Oublier plt.close() dans une boucle
```python
for col in colonnes:
    fig, ax = plt.subplots()
    ax.hist(df[col])
    fig.savefig(f"{col}.png")
    # ❌ sans close → accumulation mémoire → crash sur 100+ figures

    plt.close(fig)   # ✅ libère la mémoire après chaque figure
```

## 🪤 Piège 9 — Pie chart avec trop de catégories
```python
ax.pie(valeurs, labels=labels)  # ❌ illisible avec > 5 catégories

# ✅ Préférer un barplot horizontal
sns.barplot(data=df, y="categorie", x="valeur",
    order=df["categorie"].value_counts().index, ax=ax)
```

## 🪤 Piège 10 — Colormap jet/rainbow
```python
ax.scatter(x, y, c=vals, cmap="jet")      # ❌ trompeuse pour daltoniens
ax.scatter(x, y, c=vals, cmap="viridis")  # ✅ perceptuellement uniforme
sns.set_palette("colorblind")             # ✅ pour les catégories
```

## 🪤 Piège 11 — ax.bar() au lieu de sns.barplot() pour des moyennes
```python
# ❌ ax.bar() affiche les valeurs brutes sans IC
moyennes = df.groupby("ville")["salaire"].mean()
ax.bar(moyennes.index, moyennes.values)

# ✅ sns.barplot() calcule la moyenne ET l'IC automatiquement
sns.barplot(data=df, x="ville", y="salaire", ax=ax)
```

## 🪤 Piège 12 — figure-level Seaborn avec ax=
```python
# displot, catplot, relplot, lmplot = figure-level
# Ils ne supportent PAS ax= !
fig, ax = plt.subplots()
sns.displot(data=df, x="age", ax=ax)   # ❌ ax= ignoré, nouvelle figure créée

# ✅ utiliser la version axes-level à la place
sns.histplot(data=df, x="age", ax=ax)  # ✅
```

## Récapitulatif rapide
| Piège | Solution |
|---|---|
| savefig après show | savefig AVANT show |
| bbox_inches oublié | Toujours `bbox_inches="tight"` |
| plt.title() ambigu | Toujours `ax.set_title()` |
| axes sans .flat | `axes.flat` pour itérer |
| style permanent | `with plt.style.context()` |
| ax= oublié Seaborn | Toujours passer `ax=` |
| JPEG pour graphiques | PNG ou PDF/SVG |
| close oublié en boucle | `plt.close(fig)` après chaque figure |
| Trop de catégories pie | Barplot horizontal |
| Colormap jet | `viridis` ou `colorblind` |
| ax.bar() pour moyennes | `sns.barplot()` |
| figure-level avec ax= | Utiliser la version axes-level |
