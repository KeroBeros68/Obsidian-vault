#python #generateurs #itertools #combinatoires

`itertools` — module standard de générateurs efficaces pour itérer, combiner et filtrer.

```python
import itertools
```

## Itérateurs infinis

```python
itertools.count(start=0, step=1)    # 0, 1, 2, 3, ... (infini)
itertools.cycle(iterable)           # répète en boucle : [1,2,3] → 1,2,3,1,2,3,...
itertools.repeat(obj, times=None)   # répète obj n fois (ou indéfiniment)

list(itertools.repeat(0, 5))        # [0, 0, 0, 0, 0]
list(itertools.islice(itertools.count(10, 2), 5))  # [10, 12, 14, 16, 18]
```

## Limiter et découper

```python
itertools.islice(it, stop)          # premiers n éléments
itertools.islice(it, start, stop, step)

itertools.takewhile(pred, it)       # tant que pred(x) est vrai
itertools.dropwhile(pred, it)       # saute tant que pred(x) est vrai

list(itertools.takewhile(lambda x: x < 5, [1, 3, 5, 2, 1]))  # [1, 3]
list(itertools.dropwhile(lambda x: x < 5, [1, 3, 5, 2, 1]))  # [5, 2, 1]
```

## Combiner des itérables

```python
itertools.chain(*iterables)         # concatène : chain([1,2],[3,4]) → 1,2,3,4
itertools.chain.from_iterable(it)   # aplatit un itérable d'itérables

itertools.zip_longest(*its, fillvalue=None)   # comme zip mais jusqu'au plus long

list(itertools.chain([1,2], [3,4], [5]))      # [1, 2, 3, 4, 5]
list(itertools.chain.from_iterable([[1,2],[3,4]]))  # [1, 2, 3, 4]

list(itertools.zip_longest([1,2,3], ["a","b"], fillvalue=0))
# [(1,'a'), (2,'b'), (3,0)]
```

## Combinatoires

```python
itertools.product(*iterables, repeat=1)   # produit cartésien
itertools.permutations(it, r=None)        # permutations de longueur r
itertools.combinations(it, r)             # combinaisons sans répétition
itertools.combinations_with_replacement(it, r)

list(itertools.product([0,1], repeat=3))
# [(0,0,0),(0,0,1),(0,1,0),(0,1,1),(1,0,0),(1,0,1),(1,1,0),(1,1,1)]

list(itertools.permutations("ABC", 2))
# [('A','B'),('A','C'),('B','A'),('B','C'),('C','A'),('C','B')]

list(itertools.combinations([1,2,3,4], 2))
# [(1,2),(1,3),(1,4),(2,3),(2,4),(3,4)]
```

## Grouper et filtrer

```python
# groupby — regroupe les éléments consécutifs identiques selon une clé
# ⚠️ l'itérable doit être trié par la clé avant
data = [("A",1),("A",2),("B",3),("B",4),("A",5)]
data_triée = sorted(data, key=lambda x: x[0])

for cle, groupe in itertools.groupby(data_triée, key=lambda x: x[0]):
    print(cle, list(groupe))
# A [('A', 1), ('A', 2)]
# B [('B', 3), ('B', 4)]

# compress — filtre avec un masque booléen
list(itertools.compress("ABCDE", [1,0,1,0,1]))   # ['A', 'C', 'E']

# filterfalse — inverse de filter
list(itertools.filterfalse(lambda x: x%2, range(8)))  # [0, 2, 4, 6]
```

## `accumulate`

```python
import operator

list(itertools.accumulate([1,2,3,4,5]))                    # [1, 3, 6, 10, 15] — somme cumulée
list(itertools.accumulate([1,2,3,4,5], operator.mul))      # [1, 2, 6, 24, 120] — produit cumulé
list(itertools.accumulate([3,1,4,1,5,9], max))             # [3, 3, 4, 4, 5, 9] — max cumulé
```

## `pairwise` (Python 3.10+)

```python
list(itertools.pairwise([1,2,3,4,5]))
# [(1,2),(2,3),(3,4),(4,5)] — paires consécutives
```

## Tableau de référence rapide

| Fonction | Usage court |
|----------|------------|
| `count(n)` | entiers à partir de n |
| `cycle(it)` | répétition infinie |
| `repeat(x, n)` | x répété n fois |
| `islice(it, n)` | premiers n éléments |
| `takewhile(p, it)` | tant que p vrai |
| `dropwhile(p, it)` | après que p devient faux |
| `chain(*its)` | concaténation |
| `zip_longest` | zip jusqu'au plus long |
| `product` | produit cartésien |
| `permutations` | permutations |
| `combinations` | combinaisons |
| `groupby` | regrouper par clé |
| `accumulate` | réduction cumulative |
| `pairwise` | paires consécutives |
