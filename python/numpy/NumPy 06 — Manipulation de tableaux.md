#numpy #reshape #concatenate #stack

## Aplatir

```python
m.flatten()   # → 1D, toujours une COPIE
m.ravel()     # → 1D, VUE si possible (plus rapide)
```

## Concatenate

```python
np.concatenate([a, b], axis=0)  # empile verticalement
np.concatenate([a, b], axis=1)  # colle horizontalement
```

## vstack / hstack — raccourcis

```python
np.vstack([a, b])   # = concatenate axis=0
np.hstack([a, b])   # = concatenate axis=1
```

## stack — nouvelle dimension

```python
# a et b de shape (3,)
np.stack([a, b], axis=0)  # → (2, 3)  les arrays deviennent des lignes
np.stack([a, b], axis=1)  # → (3, 2)  les arrays deviennent des colonnes
```

> [!tip] concatenate vs stack `concatenate` → assemble sur un axe **existant** `stack` → crée un axe **nouveau**

## split

```python
np.split(a, 3)         # 3 morceaux égaux
np.split(a, [3, 7])    # aux indices 3 et 7
np.vsplit(m, 2)        # coupe horizontalement
np.hsplit(m, 3)        # coupe verticalement
```

## Transpose

```python
m.T         # inverse les axes, toujours une VUE
# (m, n) → (n, m)
```
