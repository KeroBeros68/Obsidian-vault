#python #bases #compréhensions #list-comprehension

## Compréhension de liste

```python
# Forme : [expression for element in iterable if condition]

carrés = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

pairs = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

mots_longs = [m.upper() for m in ["chat","éléphant","rat"] if len(m) > 3]
# ['ÉLÉPHANT']
```

Équivalent boucle for :
```python
# Boucle
result = []
for x in range(10):
    if x % 2 == 0:
        result.append(x**2)

# Compréhension ✅
result = [x**2 for x in range(10) if x % 2 == 0]
```

## Compréhension de dictionnaire

```python
# Forme : {cle: valeur for element in iterable}

carrés = {n: n**2 for n in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

inversé = {v: k for k, v in {"a": 1, "b": 2}.items()}
# {1: "a", 2: "b"}

filtré = {k: v for k, v in scores.items() if v >= 10}
```

## Compréhension d'ensemble

```python
unique_longueurs = {len(m) for m in ["chat","rat","chien","rat"]}
# {3, 4, 5}
```

## Expression génératrice

Comme une compréhension de liste, mais **paresseuse** — ne calcule qu'à la demande.

```python
gen = (x**2 for x in range(1_000_000))   # aucun calcul ici
next(gen)    # 0 — premier élément calculé
next(gen)    # 1

# Consomme peu de mémoire — idéal pour de grands volumes
total = sum(x**2 for x in range(1_000_000))   # ✅ pas de liste en mémoire
```

## Compréhensions imbriquées

```python
# Aplatir une matrice
matrice = [[1,2,3],[4,5,6],[7,8,9]]
plat = [val for ligne in matrice for val in ligne]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Produit cartésien
paires = [(x, y) for x in [1,2,3] for y in ["a","b"]]
# [(1,'a'), (1,'b'), (2,'a'), (2,'b'), (3,'a'), (3,'b')]
```

Lire de gauche à droite dans l'ordre des boucles imbriquées.

## Conditionnel dans l'expression

```python
# if/else dans l'expression (pas un filtre — toujours une valeur produite)
labels = ["pair" if x % 2 == 0 else "impair" for x in range(5)]
# ['pair', 'impair', 'pair', 'impair', 'pair']

# if à la fin = filtre (éléments exclus)
pairs = [x for x in range(10) if x % 2 == 0]
```

## Comparaison des formes

| Syntaxe | Résultat | Mémoire |
|---------|----------|---------|
| `[x for x in ...]` | `list` | Allouée d'un coup |
| `{x for x in ...}` | `set` | Allouée d'un coup |
| `{k:v for ...}` | `dict` | Allouée d'un coup |
| `(x for x in ...)` | `generator` | Paresseuse — O(1) |

> [!tip] Générateur dans `sum`, `max`, `min`, `any`, `all`
> Ces fonctions acceptent un itérable — passer une expression génératrice évite de construire la liste intermédiaire :
> ```python
> any(x > 10 for x in liste)   # ✅ court-circuite dès le premier True
> all(x > 0 for x in liste)    # ✅
> ```

> [!warning] Compréhension trop complexe
> Si la compréhension dépasse une ligne lisible, la remplacer par une boucle for avec un nom de variable explicite. La lisibilité prime.
