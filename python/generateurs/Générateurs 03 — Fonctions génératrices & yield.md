#python #generateurs #yield #lazy

## Fonction génératrice

Une fonction contenant `yield` retourne un **générateur** au lieu d'exécuter son corps.

```python
def compteur(debut, fin):
    while debut < fin:
        yield debut          # suspend l'exécution, retourne la valeur
        debut += 1           # reprend ici au prochain next()

gen = compteur(1, 4)        # rien n'est exécuté encore
type(gen)                   # <class 'generator'>

next(gen)   # 1 — exécute jusqu'au yield, suspend
next(gen)   # 2
next(gen)   # 3
next(gen)   # StopIteration — la fonction s'est terminée

list(compteur(0, 5))   # [0, 1, 2, 3, 4]
```

## Mécanique de `yield`

```
Appel next()
    │
    ▼
Exécution reprend après le dernier yield (ou au début)
    │
    ▼
Atteint yield valeur  →  suspend, retourne valeur à l'appelant
    │
    ▼
Attente du prochain next()
```

Quand la fonction se termine (ou atteint `return`), `StopIteration` est levée.

## `return` dans un générateur

```python
def gen():
    yield 1
    yield 2
    return "terminé"    # valeur portée par StopIteration (rarement utilisée directement)
    yield 3             # jamais atteint

list(gen())   # [1, 2]

# Récupérer la valeur de return :
g = gen()
try:
    while True: next(g)
except StopIteration as e:
    e.value   # "terminé"
```

## Fibonacci — comparaison classe vs générateur

```python
# Classe itérateur (fiche 02) — verbeux
class Fibonacci:
    def __init__(self): self.a, self.b = 0, 1
    def __iter__(self): return self
    def __next__(self):
        val = self.a
        self.a, self.b = self.b, self.a + self.b
        return val

# Fonction génératrice — idiomatique ✅
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

import itertools
list(itertools.islice(fibonacci(), 8))   # [0, 1, 1, 2, 3, 5, 8, 13]
```

## Évaluation paresseuse (lazy)

Le corps n'est exécuté **qu'à la demande**, valeur par valeur.

```python
def lire_gros_fichier(chemin):
    with open(chemin) as f:
        for ligne in f:
            yield ligne.strip()   # une ligne en mémoire à la fois

for ligne in lire_gros_fichier("data.csv"):
    traiter(ligne)   # ✅ pas de chargement total du fichier
```

## Pipeline de générateurs

Les générateurs se chaînent naturellement — chaque étape est paresseuse.

```python
def entiers():
    n = 0
    while True:
        yield n
        n += 1

def pairs(gen):
    for n in gen:
        if n % 2 == 0:
            yield n

def carres(gen):
    for n in gen:
        yield n ** 2

import itertools
pipeline = carres(pairs(entiers()))
list(itertools.islice(pipeline, 5))   # [0, 4, 16, 36, 64]
```

> [!tip] Un générateur = un itérateur à usage unique
> Contrairement à une liste, un générateur ne peut pas être parcouru deux fois. `list(gen)` le consomme entièrement.
> ```python
> gen = (x**2 for x in range(5))
> list(gen)   # [0, 1, 4, 9, 16]
> list(gen)   # []  — épuisé
> ```
