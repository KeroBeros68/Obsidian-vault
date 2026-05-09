#pandas #filtrage #apply #map #transformation

## Filtrage

python

```python
df[df["age"] > 28]
df[(df["age"] > 25) & (df["salaire"] > 3500)]
df[df["ville"].isin(["Paris","Lyon"])]
df[~df["ville"].isin(["Paris","Lyon"])]   # exclusion
```

## apply — transformer une colonne

python

```python
df["age"].apply(lambda x: x * 2)
df["nom"].apply(lambda x: x.upper())

def categorise(age):
    if age < 25:  return "junior"
    if age < 35:  return "senior"
    return "expert"

df["cat"] = df["age"].apply(categorise)
```

## apply sur les lignes — axis=1

python

```python
df["ratio"] = df.apply(lambda row: row["salaire"]/row["age"], axis=1)
```

> [!warning] apply est lent Pour les opérations simples → toujours vectoriser :
> 
> python
> 
> ```python
> df["double"] = df["age"] * 2          # ✅ rapide
> df["double"] = df["age"].apply(lambda x: x*2)  # ❌ lent
> ```

## map — remplacer via dictionnaire

python

```python
df["code"] = df["ville"].map({"Paris":75,"Lyon":69,"Bordeaux":33})
```

## replace

python

```python
df["ville"].replace("Lyon","LYON")
df["ville"].replace({"Paris":"PAR","Lyon":"LYO"})
```

## np.where — if/else vectorisé

python

```python
df["bonus"] = np.where(df["salaire"] > 4000, df["salaire"]*0.1, 0)
```

## np.select — conditions multiples

python

```python
conditions = [(df["age"]<25)&(df["salaire"]>3000),
              (df["age"]>35)&(df["salaire"]<3000)]
choix = ["jeune_bien_paye","senior_sous_paye"]
df["cat"] = np.select(conditions, choix, default="standard")
```

## pd.cut — discrétiser

python

```python
df["tranche"] = pd.cut(df["age"], bins=[0,25,35,100],
                       labels=["junior","senior","expert"])
```

## query — filtres lisibles

python

```python
df.query("age > 25 and ville == 'Paris'")
seuil = 3500
df.query("salaire > @seuil")   # @ pour les variables Python
```

## assign — ajouter des colonnes proprement

python

```python
df = (df
    .assign(salaire_net=lambda d: d["salaire"]*0.75)
    .assign(senior=lambda d: d["age"]>=30)
)
```
