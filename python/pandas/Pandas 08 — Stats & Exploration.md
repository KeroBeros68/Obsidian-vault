#python #pandas #eda #statistiques #exploration #corrélation

## Séquence EDA systématique


```python
df.shape
df.dtypes
df.head()
df.info()
df.describe()
df.isnull().sum()
```

## describe — stats descriptives


```python
df.describe()          # colonnes numériques
df.describe(include="object")  # colonnes texte
```

Lignes : count, mean, std, min, 25%, **50% (médiane)**, 75%, max

## Corrélations


```python
df.corr()                        # matrice de corrélation
df["age"].corr(df["salaire"])    # entre deux colonnes

# Interprétation :
# |r| > 0.7  → forte
# 0.3-0.7    → modérée
# < 0.3      → faible
```

## Détecter les outliers — méthode IQR


```python
Q1  = df["col"].quantile(0.25)
Q3  = df["col"].quantile(0.75)
IQR = Q3 - Q1
borne_inf = Q1 - 1.5 * IQR
borne_sup = Q3 + 1.5 * IQR   # ← borne supérieure
outliers = df[(df["col"] < borne_inf) | (df["col"] > borne_sup)]
```

## Fonctions utiles


```python
df["col"].nunique()         # nb valeurs distinctes
df["col"].unique()          # liste des valeurs
df["col"].value_counts()    # fréquences
df["col"].skew()            # asymétrie
df["col"].kurt()            # aplatissement
df["col"].quantile(0.9)     # 90e percentile
df.sample(100, random_state=42)  # échantillon reproductible
```

## crosstab — tableau de contingence


```python
pd.crosstab(df["ville"], df["senior"])
pd.crosstab(df["ville"], df["senior"], normalize="index")  # proportions
```
