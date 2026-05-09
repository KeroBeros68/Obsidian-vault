#numpy #shape #dtype #dimensions

## Attributs essentiels

```python
a.ndim    # nombre de dimensions
a.shape   # tuple (lignes, colonnes, ...)
a.size    # nombre total d'éléments
a.dtype   # type des données
```

## Dimensions

|ndim|Type|Exemple shape|
|---|---|---|
|1|Vecteur|`(5,)`|
|2|Matrice|`(3, 4)`|
|3|Tenseur|`(2, 3, 4)`|

> [!tip] Astuce mémo ndim = nombre de crochets `[` imbriqués au début du tableau

## dtype courants

|dtype|Taille|Usage|
|---|---|---|
|`float64`|8 octets|défaut pour les flottants|
|`float32`|4 octets|ML, GPU|
|`int64`|8 octets|défaut pour les entiers|
|`int32`|4 octets|économie mémoire|
|`bool`|1 octet|masques booléens|

## Upcasting automatique

```python
np.array([1, 2, 3.0])   # → float64 (remonte vers le + général)
```

## reshape

```python
a.reshape(3, 4)    # shape (3, 4), total doit rester identique
a.reshape(3, -1)   # NumPy calcule la 2e dimension auto
a.reshape(-1)      # aplatit en 1D
```

## Changer le dtype

```python
a.astype(float)       # → float64
a.astype(np.int32)    # → int32
```
