#pandas #machine-learning #preprocessing #encodage #split

## Pipeline de préparation

```
1. Nettoyer (NaN, doublons, types)
2. Encoder les catégorielles
3. Séparer X et y
4. Train/Test split
5. Normaliser (sur train seulement !)
```

## Encodage des catégorielles

python

```python
# Label encoding (ordinales)
df["cat"] = df["cat"].map({"junior":0,"senior":1,"expert":2})

# One-Hot Encoding (nominales)
df = pd.get_dummies(df, columns=["ville"], drop_first=True)
# drop_first=True évite la multicolinéarité
```

## Séparer X et y

python

```python
y = df["salaire"]
X = df.drop(columns=["salaire"])
```

## Train / Test split

python

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

> [!warning] Ne jamais normaliser AVANT le split → Data leakage ! Les stats du test contaminent l'entraînement.

## Normalisation

python

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)  # fit + transform sur train
X_test_s  = scaler.transform(X_test)       # transform SEULEMENT sur test
```

|Scaler|Usage|
|---|---|
|`StandardScaler`|SVM, régression, PCA (défaut)|
|`MinMaxScaler`|Réseaux de neurones, KNN|

## Imputation

python

```python
from sklearn.impute import SimpleImputer
imputer = SimpleImputer(strategy="median")
X_train_imp = imputer.fit_transform(X_train)
X_test_imp  = imputer.transform(X_test)
```

## Feature Engineering dates

python

```python
df["date"] = pd.to_datetime(df["date"])
df["mois"]        = df["date"].dt.month
df["jour_semaine"]= df["date"].dt.dayofweek
df["annee"]       = df["date"].dt.year
```

## Vérifications finales

python

```python
assert X_train.isnull().sum().sum() == 0
assert X_train.select_dtypes(include="object").empty
assert X_train.shape[1] == X_test.shape[1]
```
