#python #pandas #series #dataframe #bases

## Import


```python
import pandas as pd
import numpy as np
```

## Series — tableau 1D avec index


```python
s = pd.Series([10, 20, 30], index=["a","b","c"])
s["b"]    # 20 — accès par étiquette
s[0]      # 10 — accès par position
```

Depuis un dictionnaire :


```python
s = pd.Series({"Paris": 2_000_000, "Lyon": 500_000})
```

## DataFrame — tableau 2D


```python
df = pd.DataFrame({
    "nom":   ["Alice", "Bob"],
    "age":   [25, 32],
    "ville": ["Paris", "Lyon"]
})
```

## Attributs essentiels

|Attribut|Description|
|---|---|
|`df.shape`|(lignes, colonnes)|
|`df.dtypes`|type de chaque colonne|
|`df.columns`|noms des colonnes|
|`df.index`|étiquettes des lignes|
|`df.values`|array NumPy sous-jacent|
|`df.size`|nb total de cellules|
|`df.ndim`|toujours 2|

## Exploration rapide


```python
df.head(5)      # 5 premières lignes
df.tail(3)      # 3 dernières lignes
df.info()       # résumé complet
df.describe()   # stats descriptives
```

## Accéder aux colonnes


```python
df["age"]           # → Series  (1 crochet)
df[["nom","age"]]   # → DataFrame (2 crochets)
df.age              # notation pointée (fragile)
```

## Ajouter / supprimer des colonnes


```python
df["senior"] = df["age"] > 30              # ajouter
df.drop(columns=["senior"], inplace=True)  # supprimer
```

> [!tip] Series vs array NumPy Une Series possède un **index avec étiquettes** — un array NumPy non. C'est ce qui permet l'alignement automatique des données.
