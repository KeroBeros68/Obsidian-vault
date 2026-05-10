#python #pandas #loc #iloc #sélection #indexation

## Les trois façons de sélectionner

|Méthode|Type|Par|
|---|---|---|
|`df["col"]`|colonne|nom|
|`df.loc`|lignes + colonnes|**étiquette**|
|`df.iloc`|lignes + colonnes|**position**|

> [!tip] Mémo **`loc`** → **L**abel | **`iloc`** → **i**nteger

## iloc — par position


```python
df.iloc[0]           # 1ère ligne
df.iloc[-1]          # dernière ligne
df.iloc[1:3]         # lignes 1 et 2 (stop EXCLU)
df.iloc[[0,2]]       # lignes 0 et 2
df.iloc[0, 1]        # ligne 0, colonne 1
df.iloc[1:3, 0:2]    # sous-tableau
df.iloc[:, 1]        # toute la colonne 1
```

## loc — par étiquette


```python
df.loc[0]                    # ligne index 0
df.loc[1:3]                  # lignes 1, 2 ET 3 (stop INCLUS !)
df.loc[0, "age"]             # cellule précise
df.loc[:, "age"]             # toute la colonne "age"
df.loc[1:2, "nom":"ville"]   # plage de colonnes
```

> [!warning] Différence cruciale `iloc[1:3]` → lignes 1, 2 (stop **exclu**) `loc[1:3]` → lignes 1, 2, 3 (stop **inclus**)

## Sélection conditionnelle


```python
df.loc[df["age"] > 28]
df.loc[df["age"] > 28, "nom"]
df.loc[(df["age"] > 25) & (df["ville"] == "Paris")]
```

## Modifier avec loc


```python
df.loc[0, "age"] = 26
df.loc[df["ville"] == "Paris", "age"] = 99
```

> [!warning] SettingWithCopyWarning Toujours modifier via `df.loc[condition, col] = val` Jamais `df[condition]["col"] = val` → modification ignorée !

## Accès rapide à une cellule


```python
df.at[0, "age"]    # par étiquette (rapide)
df.iat[0, 1]       # par position  (rapide)
```
