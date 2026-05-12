#python #generateurs #iterateurs #classes

## Le protocole itérateur

Implémenter `__iter__` et `__next__` sur une classe pour la rendre itérable.

```python
class Compteur:
    def __init__(self, debut, fin):
        self.courant = debut
        self.fin = fin

    def __iter__(self):
        return self          # l'objet est lui-même son propre itérateur

    def __next__(self):
        if self.courant >= self.fin:
            raise StopIteration
        val = self.courant
        self.courant += 1
        return val

for n in Compteur(1, 5):
    print(n)   # 1 2 3 4

list(Compteur(0, 3))   # [0, 1, 2]
```

## Séparer itérable et itérateur

Pour rendre un objet réutilisable (plusieurs `for` indépendants), séparer la classe itérable de son itérateur.

```python
class Plage:
    """Itérable — peut créer plusieurs itérateurs indépendants."""
    def __init__(self, debut, fin):
        self.debut = debut
        self.fin = fin

    def __iter__(self):
        return PlageIterateur(self.debut, self.fin)   # nouvel itérateur à chaque fois


class PlageIterateur:
    """Itérateur — état propre à une traversée."""
    def __init__(self, debut, fin):
        self.courant = debut
        self.fin = fin

    def __iter__(self):
        return self

    def __next__(self):
        if self.courant >= self.fin:
            raise StopIteration
        val = self.courant
        self.courant += 1
        return val

p = Plage(0, 3)
list(p)    # [0, 1, 2]
list(p)    # [0, 1, 2] — ✅ réutilisable, contrairement à un itérateur direct
```

## Itérateur infini

```python
class Fibonacci:
    def __init__(self):
        self.a, self.b = 0, 1

    def __iter__(self):
        return self

    def __next__(self):
        val = self.a
        self.a, self.b = self.b, self.a + self.b
        return val

fib = Fibonacci()
[next(fib) for _ in range(8)]   # [0, 1, 1, 2, 3, 5, 8, 13]
```

Pas de `StopIteration` — infini. Utiliser `itertools.islice` pour limiter :

```python
import itertools
list(itertools.islice(Fibonacci(), 10))   # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

## Itérateur inversé — `__reversed__`

```python
class Tableau:
    def __init__(self, data):
        self.data = data

    def __iter__(self):
        return iter(self.data)

    def __reversed__(self):
        return iter(reversed(self.data))   # ✅ support de reversed()

t = Tableau([1, 2, 3])
list(reversed(t))   # [3, 2, 1]
```

> [!tip] En pratique : préférer les générateurs
> Implémenter `__iter__` / `__next__` sur une classe est verbeux. Dans la plupart des cas, une **fonction génératrice** (fiche 03) fait la même chose en beaucoup moins de lignes.
> Réserver les classes itérateurs pour les cas où un état complexe ou une réutilisabilité fine est nécessaire.
