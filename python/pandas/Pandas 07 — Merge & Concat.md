#python #pandas #merge #concat #join #sql

## concat — empiler des DataFrames


```python
pd.concat([df1, df2], ignore_index=True)   # vertical (lignes)
pd.concat([df1, df2], axis=1)              # horizontal (colonnes)
```

> [!warning] Toujours ignore_index=True Sans ça, l'index peut avoir des doublons !

## merge — jointure sur une clé


```python
pd.merge(clients, commandes, left_on="id", right_on="id_client")
pd.merge(df1, df2, on="id")               # même nom de clé
```

## Types de jointures — how

|how|Comportement|
|---|---|
|`inner`|intersection (défaut)|
|`left`|tout à gauche + NaN à droite|
|`right`|tout à droite + NaN à gauche|
|`outer`|tout + NaN partout|


```python
pd.merge(df1, df2, on="id", how="left")
```

> [!tip] Left join Le plus utilisé en pratique — garantit que toutes les lignes de gauche sont conservées.

## Colonnes dupliquées — suffixes


```python
pd.merge(df1, df2, on="id")
# → valeur_x, valeur_y  (suffixes automatiques)

pd.merge(df1, df2, on="id", suffixes=("_2023","_2024"))
# → valeur_2023, valeur_2024
```

## join — jointure sur l'index


```python
df1.join(df2)   # plus rapide si l'index est déjà la clé
```
