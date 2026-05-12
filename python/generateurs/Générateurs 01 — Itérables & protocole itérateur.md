#python #generateurs #iterables #protocole

## Itérable vs itérateur

| | Itérable | Itérateur |
|---|---|---|
| Définition | Peut produire un itérateur | Produit les valeurs une à une |
| Méthode clé | `__iter__()` | `__iter__()` + `__next__()` |
| Exemples | `list`, `str`, `dict`, `range` | `map`, `zip`, `enumerate`, générateur |
| Réutilisable | ✅ (crée un nouvel itérateur à chaque fois) | ❌ (épuisable) |

```python
lst = [1, 2, 3]   # itérable — pas un itérateur

it = iter(lst)    # crée un itérateur depuis l'itérable
next(it)          # 1
next(it)          # 2
next(it)          # 3
next(it)          # StopIteration ❌ — épuisé
```

## `iter()` et `next()`

```python
iter(obj)          # appelle obj.__iter__() — retourne un itérateur
next(it)           # appelle it.__next__() — retourne la valeur suivante
next(it, defaut)   # retourne defaut au lieu de lever StopIteration ✅
```

```python
it = iter([10, 20, 30])

while True:
    val = next(it, None)   # None quand épuisé
    if val is None:
        break
    print(val)
```

## Ce que fait `for` en coulisses

```python
for x in [1, 2, 3]:
    print(x)

# Équivalent exact :
_it = iter([1, 2, 3])
while True:
    try:
        x = next(_it)
    except StopIteration:
        break
    print(x)
```

`for` appelle `iter()` sur l'objet, puis `next()` en boucle jusqu'à `StopIteration`.

## `StopIteration`

Signal de fin d'itération. Levé par `__next__` quand il n'y a plus de valeurs.

```python
it = iter([])
next(it)              # StopIteration immédiat

next(it, "fini")      # "fini" — version sans exception ✅
```

## Un itérateur est aussi un itérable

Tout itérateur implémente `__iter__` (qui retourne `self`). Il peut donc s'utiliser dans un `for`.

```python
it = iter([1, 2, 3])
for x in it:          # ✅ — iter(it) retourne it lui-même
    print(x)

# Mais épuisé après :
for x in it:
    print(x)          # rien — l'itérateur est à sec
```

> [!tip] Différencier avec `hasattr`
> ```python
> hasattr(obj, '__iter__')                    # itérable ?
> hasattr(obj, '__iter__') and hasattr(obj, '__next__')  # itérateur ?
> ```
