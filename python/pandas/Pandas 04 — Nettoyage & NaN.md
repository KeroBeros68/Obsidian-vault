#python #pandas #nan #nettoyage #missing-values

## Détecter les NaN


```python
df.isnull()                        # DataFrame de True/False
df.isnull().sum()                  # nb de NaN par colonne
df.isnull().sum() / len(df) * 100  # % de NaN
df.isnull().any()                  # au moins 1 NaN par colonne
```

## Supprimer les NaN — dropna


```python
df.dropna()                     # toute ligne avec ≥1 NaN
df.dropna(axis=1)               # toute colonne avec ≥1 NaN
df.dropna(how="all")            # seulement si TOUTE la ligne est NaN
df.dropna(subset=["age"])       # seulement si NaN dans "age"
df.dropna(thresh=2)             # garde si ≥2 valeurs non-NaN
```

## Remplir les NaN — fillna


```python
df.fillna(0)
df["age"].fillna(df["age"].median())    # médiane recommandée en ML
df["ville"].fillna("Inconnu")
df.fillna(method="ffill")              # propage valeur précédente
df.fillna(method="bfill")              # propage valeur suivante
df.fillna({"age": df["age"].median(), "ville": "Inconnu"})
```

> [!tip] Bonne pratique ML Numérique → **médiane** (robuste aux outliers) Catégoriel → **mode** ou `"Inconnu"`

## Doublons


```python
df.duplicated().sum()
df.drop_duplicates()
df.drop_duplicates(subset=["nom"])
df.drop_duplicates(keep="last")
```

## Convertir les types


```python
df["age"]  = df["age"].astype(int)
df["date"] = pd.to_datetime(df["date"])
df["age"]  = pd.to_numeric(df["age"], errors="coerce")  # NaN si impossible
```

## Nettoyer le texte


```python
df["nom"].str.strip()          # espaces
df["nom"].str.lower()          # minuscules
df["nom"].str.replace("-"," ") # remplacer
```

## Après nettoyage


```python
df = df.dropna().reset_index(drop=True)  # toujours reset_index !
```
