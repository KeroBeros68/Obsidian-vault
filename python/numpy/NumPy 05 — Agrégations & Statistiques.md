#python #numpy #agrégations #statistiques #axis

## Fonctions de base

```python
np.sum(a)       # somme
np.min(a)       # minimum
np.max(a)       # maximum
np.mean(a)      # moyenne
np.median(a)    # médiane
np.std(a)       # écart-type
np.var(a)       # variance
```

Aussi disponibles comme méthodes : `a.sum()`, `a.mean()`, etc.

## Le paramètre axis

> [!tip] Mémo axis `axis=0` → efface les **lignes** → résultat par colonne ↓ `axis=1` → efface les **colonnes** → résultat par ligne →

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

np.sum(m)           # 21  (tout)
np.sum(m, axis=0)   # [5  7  9]   par colonne
np.sum(m, axis=1)   # [6  15]     par ligne
```

## argmin / argmax — indice du min/max

```python
np.argmax(a)   # indice de la valeur max
np.argmin(a)   # indice de la valeur min
```

## Fonctions utiles

```python
np.sort(a)              # trié (copie)
np.unique(a)            # valeurs uniques
np.cumsum(a)            # somme cumulative
np.percentile(a, 75)    # 75e percentile
np.clip(a, min, max)    # borne les valeurs entre min et max
```
