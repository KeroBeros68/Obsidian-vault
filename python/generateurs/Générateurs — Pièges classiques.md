#python #generateurs #pièges #erreurs #debugging

## 🪤 Piège 1 — Générateur épuisé réutilisé

```python
gen = (x**2 for x in range(5))
list(gen)   # [0, 1, 4, 9, 16]
list(gen)   # [] — épuisé silencieusement ❌

# ✅ si besoin de parcourir plusieurs fois, matérialiser en liste
data = list(x**2 for x in range(5))
```

---

## 🪤 Piège 2 — `send()` sans amorçage

```python
def gen():
    val = yield
    yield val * 2

g = gen()
g.send(10)   # TypeError — impossible d'envoyer une valeur non-None à un générateur fraîchement créé ❌

g = gen()
next(g)       # amorçage ✅
g.send(10)    # 20
```

---

## 🪤 Piège 3 — `return` dans un générateur ne retourne pas une valeur directement

```python
def gen():
    yield 1
    return 42   # ne retourne pas 42 à l'appelant du générateur

g = gen()
next(g)   # 1
next(g)   # StopIteration — 42 est dans e.value, pas retourné directement

# Pour récupérer la valeur de return :
try:
    while True: next(g)
except StopIteration as e:
    print(e.value)   # 42
```

---

## 🪤 Piège 4 — Capturer `GeneratorExit`

```python
def gen():
    try:
        while True:
            yield
    except GeneratorExit:
        yield "encore"   # ❌ RuntimeError — ne pas yield après GeneratorExit

def gen():
    try:
        while True:
            yield
    except GeneratorExit:
        print("nettoyage")   # ✅ — nettoyage sans yield
        # ou simplement : finally: print("nettoyage")
```

---

## 🪤 Piège 5 — `groupby` sans tri préalable

```python
data = [("B",2), ("A",1), ("B",3), ("A",4)]

for cle, grp in itertools.groupby(data, key=lambda x: x[0]):
    print(cle, list(grp))
# B [('B', 2)]
# A [('A', 1)]
# B [('B', 3)]   ← B apparaît deux fois ❌ — groupby ne regroupe que le consécutif

# ✅ trier d'abord
for cle, grp in itertools.groupby(sorted(data, key=lambda x: x[0]), lambda x: x[0]):
    print(cle, list(grp))
```

---

## 🪤 Piège 6 — Itérer sur un dict en le modifiant

```python
d = {"a": 1, "b": 2, "c": 3}
for k in d:
    del d[k]   # ❌ RuntimeError: dictionary changed size during iteration

for k in list(d):       # ✅ copier les clés
    del d[k]
```

---

## 🪤 Piège 7 — Variable de boucle capturée par le générateur

```python
gens = [x**i for i in range(3) for x in [2]]  # liste — OK, évaluation immédiate

# Avec des générateurs dans une liste :
gens = [(lambda: i**2) for i in range(3)]
[g() for g in gens]   # [4, 4, 4] — tous capturent i=2 ❌

gens = [(lambda i=i: i**2) for i in range(3)]
[g() for g in gens]   # [0, 1, 4] ✅ — capturer la valeur
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Générateur réutilisé | `list(gen)` pour matérialiser |
| `send()` sans amorçage | `next(gen)` avant le premier `send` |
| `return` dans un générateur | Valeur dans `StopIteration.value` |
| `yield` après `GeneratorExit` | `finally` sans `yield` |
| `groupby` sans tri | `sorted()` avant `groupby` |
| Modifier un dict en itérant | Itérer sur `list(d)` |
| Closure capturant une variable de boucle | Capturer avec un argument par défaut `i=i` |
