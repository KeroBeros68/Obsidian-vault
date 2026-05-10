#python #pandas #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier index=False dans to_csv


```python
df.to_csv("data.csv")              # ❌ colonne Unnamed:0 au prochain import
df.to_csv("data.csv", index=False) # ✅
```

## 🪤 Piège 2 — Un crochet vs deux crochets


```python
df["age"]     # → Series    (une colonne)
df[["age"]]   # → DataFrame (un tableau à 1 colonne)
```

> [!warning] Beaucoup de fonctions sklearn attendent un **DataFrame** — utiliser `[[]]` si nécessaire.

## 🪤 Piège 3 — loc inclut le stop, iloc non


```python
df.loc[1:3]    # lignes 1, 2, 3  (stop INCLUS)
df.iloc[1:3]   # lignes 1, 2     (stop EXCLU)
```

## 🪤 Piège 4 — SettingWithCopyWarning


```python
df[df["age"] > 30]["nom"] = "X"              # ❌ modification ignorée
df.loc[df["age"] > 30, "nom"] = "X"          # ✅ toujours utiliser loc
```

## 🪤 Piège 5 — Normaliser avant le split → Data Leakage


```python
# ❌ Les stats du test contaminent l'entraînement
scaler.fit_transform(X)

# ✅ Toujours : split → fit sur train → transform les deux
X_train, X_test = train_test_split(X)
scaler.fit(X_train)
X_train_s = scaler.transform(X_train)
X_test_s  = scaler.transform(X_test)
```

## 🪤 Piège 6 — dropna retourne une copie


```python
df.dropna()                # ❌ n'affecte pas df
df = df.dropna()           # ✅ réassigner
df.dropna(inplace=True)    # ✅ ou inplace
```

## 🪤 Piège 7 — Oublier reset_index après dropna/filter


```python
df = df.dropna().reset_index(drop=True)  # ✅
# Sans reset_index → trous dans l'index → bugs avec iloc
```

## 🪤 Piège 8 — apply au lieu de vectorisation


```python
df["double"] = df["age"].apply(lambda x: x*2)  # ❌ lent
df["double"] = df["age"] * 2                    # ✅ 100x plus rapide
```

## 🪤 Piège 9 — iterrows sur un gros dataset


```python
for i, row in df.iterrows():   # ❌ catastrophiquement lent
    ...
df["col"] = df["a"] * df["b"]  # ✅ vectorisé
```

## 🪤 Piège 10 — assign sans lambda dans un chaînage


```python
# ❌ df référence l'original, pas le df transformé dans la chaîne
df.dropna().assign(bonus=df["salaire"] * 0.1)

# ✅ lambda d: référence le df courant dans la chaîne
df.dropna().assign(bonus=lambda d: d["salaire"] * 0.1)
```

## 🪤 Piège 11 — savefig après show


```python
plt.show()           # ❌ vide le buffer → image vide
plt.savefig("g.png") # ← sauvegarde une image vide !

plt.savefig("g.png") # ✅ sauvegarder AVANT
plt.show()
```

## 🪤 Piège 12 — concat sans ignore_index


```python
pd.concat([df1, df2])                      # ❌ index dupliqués
pd.concat([df1, df2], ignore_index=True)   # ✅
```

## Récapitulatif rapide

|Piège|Solution|
|---|---|
|index=False oublié|Toujours ajouter à to_csv/to_excel|
|`[]` vs `[[]]`|`[]` → Series, `[[]]` → DataFrame|
|loc stop inclus|loc inclut, iloc exclut|
|SettingWithCopyWarning|Toujours modifier via loc|
|Data leakage|Split → fit train → transform les deux|
|dropna sans réassigner|`df = df.dropna()` ou inplace=True|
|reset_index oublié|Après dropna/filter/concat|
|apply au lieu de vectorisation|Vectoriser les opérations simples|
|iterrows|Bannir — vectoriser à la place|
|assign sans lambda|`lambda d:` dans les chaînages|
|savefig après show|savefig AVANT show|
|concat sans ignore_index|Toujours ignore_index=True|
