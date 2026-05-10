#python #numpy #création #bases

## Import

```python
import numpy as np
```

## Depuis une liste

```python
np.array([1, 2, 3])                  # 1D
np.array([[1, 2], [3, 4]])           # 2D
```

## Création rapide

|Fonction|Description|
|---|---|
|`np.zeros((m, n))`|Tableau de 0.0|
|`np.ones((m, n))`|Tableau de 1.0|
|`np.full((m, n), v)`|Tableau rempli de v|
|`np.eye(n)`|Matrice identité n×n|

## Séquences

```python
np.arange(start, stop, step)   # contrôle le PAS, stop exclu
np.linspace(start, stop, n)    # contrôle le NOMBRE de points
```

> [!tip] Différence clé `arange` → pas fixe | `linspace` → nombre de points fixe

## Aléatoire

```python
rng = np.random.default_rng(42)
rng.random((3, 3))           # flottants [0, 1)
rng.integers(0, 10, (3, 3))  # entiers
rng.normal(0, 1, (3, 3))     # distribution normale
```
