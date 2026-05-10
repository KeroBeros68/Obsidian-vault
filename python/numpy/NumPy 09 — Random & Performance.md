#python #numpy #random #performance #vectorisation

## Générateur moderne (recommandé)

```python
rng = np.random.default_rng(seed=42)  # seed = reproductibilité

rng.random((m, n))           # flottants uniformes [0, 1)
rng.integers(low, high, shape) # entiers
rng.normal(mu, sigma, shape)   # distribution normale
rng.uniform(low, high, shape)  # uniforme
rng.choice(array, n)           # tirage sans remise
rng.shuffle(a)                 # mélange en place
rng.permutation(a)             # copie mélangée
```

## Vues vs Copies — récapitulatif

|Opération|Résultat|
|---|---|
|`a[1:4]`|VUE|
|`a.reshape(...)`|VUE (si possible)|
|`a.T`|VUE|
|`a.ravel()`|VUE (si possible)|
|`a.copy()`|COPIE|
|`a.flatten()`|COPIE|
|`a[[0,1,2]]`|COPIE|

```python
np.shares_memory(a, b)   # vérifie si deux arrays partagent la mémoire
```

## Vectorisation — règle d'or

> [!tip] Ne jamais boucler là où NumPy peut le faire
> 
> ```python
> # ❌ Lent
> result = [x**2 for x in a]
> 
> # ✅ Jusqu'à 150x plus rapide
> result = a ** 2
> ```

## np.vectorize — pour ses propres fonctions

```python
def f(x):
    return x * 2 if x > 0 else -x

vf = np.vectorize(f)
vf(a)   # applique f sur chaque élément
```

## Optimisations mémoire

```python
# float32 = 4 octets vs float64 = 8 octets
np.zeros(1_000_000, dtype=np.float32)   # 4 Mo
np.zeros(1_000_000, dtype=np.float64)   # 8 Mo

# Opérations in-place — évite les copies
a += 1
a *= 2
```

## Checklist performance

- [ ] Utiliser des ufuncs plutôt que des boucles
- [ ] Exploiter le broadcasting plutôt que de répliquer
- [ ] Préférer les vues aux copies
- [ ] Spécifier `dtype=np.float32` pour les gros arrays ML
- [ ] Opérations in-place (`+=`, `*=`) sur les gros arrays
- [ ] Utiliser `rng = np.random.default_rng(seed)` pour la reproductibilité