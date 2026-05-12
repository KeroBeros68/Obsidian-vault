#python #generateurs #expressions #comprehension

## Syntaxe

```python
gen = (expression for element in iterable if condition)
```

Comme une compréhension de liste, mais avec `()` — retourne un **générateur**, pas une liste.

```python
carrés = (x**2 for x in range(10))
type(carrés)    # <class 'generator'>

next(carrés)    # 0
next(carrés)    # 1
list(carrés)    # [4, 9, 16, 25, 36, 49, 64, 81] — les deux premiers déjà consommés
```

## Comparaison avec la compréhension de liste

```python
# Liste — calcule tout immédiatement, stocke tout en mémoire
lst = [x**2 for x in range(1_000_000)]    # ~8 Mo en mémoire

# Générateur — calcule à la demande, mémoire constante
gen = (x**2 for x in range(1_000_000))    # ~200 octets
```

| | Compréhension de liste | Expression génératrice |
|---|---|---|
| Syntaxe | `[...]` | `(...)` |
| Type | `list` | `generator` |
| Évaluation | Immédiate | Paresseuse |
| Mémoire | O(n) | O(1) |
| Réutilisable | ✅ | ❌ |
| Indexable | ✅ | ❌ |

## Utilisation dans les fonctions builtin

Les fonctions acceptant un itérable peuvent recevoir une expression génératrice **sans parenthèses supplémentaires** :

```python
sum(x**2 for x in range(10))          # ✅ pas de double ()
max(len(mot) for mot in ["a","bb","ccc"])   # 3
min(abs(x) for x in [-3, -1, 2])      # 1
any(x > 10 for x in [1, 5, 15])       # True — court-circuit dès 15
all(x > 0 for x in [1, 2, 3])         # True
",".join(str(x) for x in [1, 2, 3])   # "1,2,3"
```

## Imbrication

```python
# Produit cartésien paresseux
paires = ((x, y) for x in range(3) for y in range(3))
list(paires)
# [(0,0),(0,1),(0,2),(1,0),(1,1),(1,2),(2,0),(2,1),(2,2)]

# Aplatir un itérable d'itérables
lignes = [["a","b"], ["c","d"], ["e"]]
plat = (val for sous in lignes for val in sous)
list(plat)   # ['a', 'b', 'c', 'd', 'e']
```

## Pipeline avec `filter` et `map`

```python
# Style fonctionnel — filter et map retournent des itérateurs paresseux
pairs = filter(lambda x: x % 2 == 0, range(10))
carrés = map(lambda x: x**2, pairs)
list(carrés)   # [0, 4, 16, 36, 64]

# Style expression génératrice — plus lisible ✅
list(x**2 for x in range(10) if x % 2 == 0)
```

## Passer une expression génératrice à une fonction

```python
def traiter(gen):
    for val in gen:
        print(val)

traiter(x**2 for x in range(5))   # ✅ l'expression est l'argument

# Équivalent explicite :
traiter((x**2 for x in range(5)))
```

> [!tip] Générateur ou liste ?
> - Générateur si les données sont grandes, consommées une seule fois, ou produites par un pipeline.
> - Liste si on a besoin d'indexer, de `len()`, de parcourir plusieurs fois, ou de déboguer le contenu intermédiaire.
