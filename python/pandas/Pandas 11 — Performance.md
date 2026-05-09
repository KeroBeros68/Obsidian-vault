#pandas #performance #optimisation #mémoire #vectorisation

## Règle d'or — ne jamais itérer

python

```python
# ❌ Très lent
for i, row in df.iterrows():
    df.at[i,"bonus"] = row["salaire"] * 0.1

# ✅ Vectorisé — 100x+ plus rapide
df["bonus"] = df["salaire"] * 0.1
```

## Ordre de performance

```
Vectorisation NumPy  →  le plus rapide
query/eval           →  rapide sur gros datasets
apply (axis=0)       →  lent
apply (axis=1)       →  très lent
iterrows             →  à bannir absolument
```

## Optimiser la mémoire — dtypes

python

```python
df["age"]     = df["age"].astype(np.int16)
df["salaire"] = df["salaire"].astype(np.float32)
df["ville"]   = df["ville"].astype("category")  # si peu de valeurs uniques
```

> [!tip] Règle category Si `nunique() / len(df) < 0.5` → convertir en `category` Gain mémoire typique : 50-80%

## Chaînage de méthodes

python

```python
result = (df
    .dropna()
    .query("age > 25")
    .assign(bonus=lambda d: d["salaire"] * 0.1)  # lambda d: indispensable !
    .groupby("ville")["bonus"].mean()
    .reset_index()
)
```

## Lire efficacement

python

```python
pd.read_csv("data.csv",
    usecols=["col1","col2"],     # ne lire que les colonnes utiles
    dtype={"age": np.int16},     # éviter la détection automatique
)
```

## Checklist performance

- [ ]  Vectoriser plutôt que boucler
- [ ]  Éviter apply() pour les opérations simples
- [ ]  Utiliser np.select() pour conditions multiples
- [ ]  Convertir les répétitives en `category`
- [ ]  Downcast les types numériques
- [ ]  Spécifier dtype à l'import
- [ ]  Utiliser usecols
- [ ]  Chaîner les méthodes avec lambda
- [ ]  Sauvegarder en .parquet pour les gros fichiers
