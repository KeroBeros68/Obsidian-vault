#python #pandas #groupby #agg #transform #pivot

## Syntaxe de base


```python
df.groupby("ville")["salaire"].mean()
df.groupby("ville")["salaire"].sum()
df.groupby("ville")["salaire"].count()
```

## Grouper par plusieurs colonnes


```python
df.groupby(["ville","senior"])["salaire"].mean()
```

## agg — plusieurs agrégations


```python
df.groupby("ville")["salaire"].agg(["mean","min","max","count"])

# Noms personnalisés :
df.groupby("ville")["salaire"].agg(
    moyenne = "mean",
    minimum = "min",
    effectif = "count"
)

# Sur plusieurs colonnes :
df.groupby("ville").agg({"salaire":["mean","max"],"age":["mean","min"]})
```

## transform — garde la shape originale


```python
df["moy_ville"] = df.groupby("ville")["salaire"].transform("mean")
```

> [!tip] agg vs transform `agg` → **réduit** (1 ligne par groupe) `transform` → **conserve** la taille (1 valeur par ligne)

## filter — filtrer des groupes entiers


```python
df.groupby("ville").filter(lambda g: len(g) >= 2)
```

## pivot_table — tableau croisé dynamique


```python
df.pivot_table(
    values="salaire", index="ville",
    columns="senior", aggfunc="mean", fill_value=0
)
```

## value_counts


```python
df["ville"].value_counts()              # fréquences
df["ville"].value_counts(normalize=True) # proportions
```
