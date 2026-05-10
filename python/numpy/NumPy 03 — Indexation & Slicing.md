#python #numpy #indexation #slicing

## Indexation 1D

```python
a[0]     # premier élément
a[-1]    # dernier élément
a[-2]    # avant-dernier
```

## Slicing 1D — `[start:stop:step]`

```python
a[1:4]    # indices 1, 2, 3 (stop exclu)
a[:3]     # du début à 3 exclu
a[2:]     # de l'indice 2 à la fin
a[::2]    # un élément sur deux
a[::-1]   # tableau inversé
```

## Indexation 2D

```python
m[0, 0]     # ligne 0, colonne 0
m[1, 2]     # ligne 1, colonne 2
m[-1, -1]   # dernière ligne, dernière colonne
```

## Slicing 2D

```python
m[0, :]       # toute la ligne 0
m[:, 1]       # toute la colonne 1
m[0:2, 1:3]   # sous-matrice
m[:, ::2]     # colonnes paires
```

## Fancy indexing

```python
a[[0, 2, 4]]        # indices 0, 2 et 4
m[[0, 2], :]        # lignes 0 et 2
m[:, [0, 2]]        # colonnes 0 et 2
```

> [!warning] Vue vs Copie **Slice** → VUE (données partagées en mémoire) **Fancy indexing** → COPIE (toujours)
> 
> ```python
> b = a[1:4]        # vue — modif de b modifie a !
> b = a[1:4].copy() # copie indépendante
> b = a[[1,2,3]]    # copie (fancy indexing)
> ```
