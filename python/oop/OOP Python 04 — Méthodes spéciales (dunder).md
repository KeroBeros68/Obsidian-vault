#python #oop #dunder #méthodes-spéciales

Les méthodes spéciales (entourées de `__`) permettent à vos classes de s'intégrer avec la syntaxe Python native.

## Représentation

```python
class Vecteur:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vecteur({self.x}, {self.y})"   # REPL, logs, debug

    def __str__(self):
        return f"({self.x}, {self.y})"           # print()
```

## Opérateurs arithmétiques

```python
    def __add__(self, other):
        return Vecteur(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vecteur(self.x - other.x, self.y - other.y)

    def __mul__(self, scalaire):
        return Vecteur(self.x * scalaire, self.y * scalaire)

    def __rmul__(self, scalaire):      # scalaire * vecteur (ordre inversé)
        return self.__mul__(scalaire)

    def __neg__(self):                 # opérateur unaire -v
        return Vecteur(-self.x, -self.y)

v1 = Vecteur(1, 2)
v2 = Vecteur(3, 4)
v1 + v2    # Vecteur(4, 6)
3 * v1     # Vecteur(3, 6)
```

## Comparaison

```python
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __lt__(self, other):           # <
        return abs(self) < abs(other)

    def __le__(self, other): ...       # <=
    def __gt__(self, other): ...       # >
    def __ge__(self, other): ...       # >=
```

> [!tip] `@functools.total_ordering`
> Définir `__eq__` et un seul parmi `__lt__`, `__le__`, `__gt__`, `__ge__` — le décorateur génère les autres automatiquement.
> ```python
> from functools import total_ordering
> @total_ordering
> class MaClasse:
>     def __eq__(self, other): ...
>     def __lt__(self, other): ...
> ```

## Conteneur

```python
class Pile:
    def __init__(self):
        self._data = []

    def __len__(self):
        return len(self._data)         # len(pile)

    def __getitem__(self, index):
        return self._data[index]       # pile[0], pile[1:3]

    def __setitem__(self, index, val):
        self._data[index] = val        # pile[0] = val

    def __delitem__(self, index):
        del self._data[index]          # del pile[0]

    def __contains__(self, val):
        return val in self._data       # val in pile

    def __iter__(self):
        return iter(self._data)        # for x in pile
```

## Appelable

```python
class Multiplicateur:
    def __init__(self, facteur):
        self.facteur = facteur

    def __call__(self, x):
        return x * self.facteur

double = Multiplicateur(2)
double(5)     # 10 — l'instance s'appelle comme une fonction
callable(double)  # True
```

## Gestionnaire de contexte

```python
class GestionFichier:
    def __init__(self, nom):
        self.nom = nom

    def __enter__(self):
        self.f = open(self.nom)
        return self.f              # valeur liée à `as`

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.f.close()
        return False               # False = ne pas supprimer l'exception

with GestionFichier("data.txt") as f:
    contenu = f.read()
```

## Tableau de référence rapide

| Méthode | Déclenché par |
|---------|--------------|
| `__init__` | `Classe(...)` |
| `__repr__` | `repr(obj)`, REPL |
| `__str__` | `str(obj)`, `print(obj)` |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[key]` |
| `__setitem__` | `obj[key] = val` |
| `__contains__` | `x in obj` |
| `__iter__` | `for x in obj` |
| `__call__` | `obj(...)` |
| `__eq__` | `obj == other` |
| `__hash__` | `hash(obj)`, clé de dict |
| `__bool__` | `bool(obj)`, `if obj:` |
| `__enter__`/`__exit__` | `with obj:` |
| `__add__` | `obj + other` |
| `__del__` | destruction de l'objet (finaliseur) |
