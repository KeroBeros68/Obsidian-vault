#pandas #csv #excel #import #export

## Lire un CSV

python

```python
df = pd.read_csv("data.csv")

# Paramètres courants :
df = pd.read_csv(
    "data.csv",
    sep        = ";",           # séparateur
    encoding   = "latin-1",     # encodage
    index_col  = 0,             # colonne comme index
    nrows      = 1000,          # nb de lignes max
    usecols    = ["a","b","c"], # colonnes à lire
    dtype      = {"age": np.int16},
    parse_dates= ["date"],      # convertir en datetime
    na_values  = ["N/A","?","-"]
)
```

> [!tip] Fichiers français `sep=";"` + `encoding="latin-1"` ou `"utf-8-sig"`

## Autres formats

python

```python
pd.read_excel("data.xlsx", sheet_name="Feuil1")
pd.read_json("data.json")
pd.read_parquet("data.parquet")
pd.read_sql("SELECT * FROM table", conn)
```

## Exporter

python

```python
df.to_csv("output.csv", index=False)     # ← toujours index=False !
df.to_excel("output.xlsx", index=False)
df.to_parquet("output.parquet")
df.to_json("output.json")
```

> [!warning] index=False Sans `index=False`, Pandas écrit l'index comme colonne → colonne `Unnamed: 0` parasite au prochain import !

## Réflexe post-import

python

```python
df.shape
df.head()
df.dtypes
df.info()
df.isnull().sum()
```

## Gros fichiers

python

```python
# Lire par morceaux
for chunk in pd.read_csv("data.csv", chunksize=50_000):
    process(chunk)
```
