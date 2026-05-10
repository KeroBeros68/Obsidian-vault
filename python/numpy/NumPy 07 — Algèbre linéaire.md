#python #numpy #linalg #matriciel #dot

## `*` vs `@` — la distinction fondamentale

> [!warning] Ne pas confondre ! `a * b` → multiplication **élément par élément** `a @ b` → **produit matriciel**

```python
a * b        # élément par élément
a @ b        # produit matriciel
np.dot(a, b) # identique à a @ b
```

## Règle des dimensions

```
(m, n) @ (n, p) → (m, p)
         ↑   ↑
    doivent être égaux, disparaissent
```

## np.linalg

```python
np.linalg.det(m)         # déterminant
np.linalg.inv(m)         # matrice inverse
np.linalg.norm(v)        # norme euclidienne = √(Σ xᵢ²)
np.linalg.rank(m)        # rang
np.linalg.eig(m)         # valeurs et vecteurs propres
np.linalg.svd(m)         # décomposition SVD
np.linalg.solve(A, b)    # résout Ax = b
np.linalg.lstsq(X, y)    # moindres carrés
```

## Norme euclidienne

```python
np.linalg.norm([3, 4])   # √(3² + 4²) = √25 = 5.0
```

## Matrice identité

```python
np.eye(n)       # génère la matrice identité n×n
A @ np.eye(n)   # = A  (propriété fondamentale)
```
