#matplotlib #subplots #gridspec #layout

## plt.subplots — grille régulière
```python
fig, ax  = plt.subplots(figsize=(8,5))           # 1 graphique
fig, axes = plt.subplots(2, 3, figsize=(14,8))   # grille 2×3
fig, axes = plt.subplots(2, 2, sharex=True, sharey=True)

axes[0, 0].plot(...)   # ligne 0, colonne 0
axes[1, 2].bar(...)    # ligne 1, colonne 2
```

## Itérer sur les axes
```python
for ax, col in zip(axes.flat, df.columns):
    ax.hist(df[col], bins=20)
    ax.set_title(col)
```

> [!tip] axes.flat
> `axes.flat` itère sur tous les axes en 1D — indispensable pour les grilles 2D

## subplot_mosaic — layouts asymétriques
```python
fig, axes = plt.subplot_mosaic(
    [["haut",   "haut"  ],
     ["gauche", "droite"]],
    figsize=(12, 8)
)
axes["haut"].plot(x, y)
axes["gauche"].bar(cats, vals)
axes["droite"].scatter(x, y)
```

## GridSpec — contrôle fin
```python
import matplotlib.gridspec as gridspec
fig = plt.figure(figsize=(12, 8))
gs  = gridspec.GridSpec(2, 3)

ax1 = fig.add_subplot(gs[0, :])    # toute la 1ère ligne
ax2 = fig.add_subplot(gs[1, 0])    # ligne 1, col 0
ax3 = fig.add_subplot(gs[1, 1:])   # ligne 1, cols 1-2
```

## Double axe Y — twinx
```python
fig, ax1 = plt.subplots()
ax1.plot(x, temperatures, color="coral")
ax1.set_ylabel("Température (°C)", color="coral")

ax2 = ax1.twinx()   # même axe X, 2e axe Y à droite
ax2.bar(x, precipitations, color="steelblue", alpha=0.4)
ax2.set_ylabel("Précipitations (mm)", color="steelblue")
```

## Ratios de taille
```python
fig, axes = plt.subplots(2, 1,
    gridspec_kw={"height_ratios": [3, 1]})
```

## Espacement
```python
plt.tight_layout(pad=2.0, hspace=0.4, wspace=0.3)
fig, axes = plt.subplots(layout="constrained")  # moderne
```
