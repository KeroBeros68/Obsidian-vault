#python #oop #héritage-multiple #mro #mixins

## Héritage multiple

```python
class Volant:
    def voler(self):
        return "je vole"

class Nageant:
    def nager(self):
        return "je nage"

class Canard(Volant, Nageant):   # hérite des deux
    pass

donald = Canard()
donald.voler()    # "je vole"
donald.nager()    # "je nage"
```

## MRO — Method Resolution Order

Python résout les méthodes via l'algorithme **C3 linearization**. L'ordre est lisible via `__mro__`.

```python
class A:
    def qui(self): return "A"

class B(A):
    def qui(self): return "B"

class C(A):
    def qui(self): return "C"

class D(B, C):
    pass

D.__mro__
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

D().qui()    # "B" — premier dans le MRO qui définit qui()
```

```
     A
    / \
   B   C
    \ /
     D
```

Python cherche de gauche à droite, profondeur d'abord, sans remonter un parent avant que toutes ses branches soient explorées.

## `super()` et MRO

`super()` ne remonte pas directement au parent — il suit le MRO.

```python
class A:
    def f(self):
        print("A")

class B(A):
    def f(self):
        print("B")
        super().f()    # appelle C.f (suivant dans le MRO de D), pas A.f

class C(A):
    def f(self):
        print("C")
        super().f()    # appelle A.f

class D(B, C):
    def f(self):
        print("D")
        super().f()    # appelle B.f

D().f()   # D → B → C → A
```

Pour que `super()` fonctionne correctement en héritage multiple, **toutes les classes doivent appeler `super()`** dans la méthode concernée.

## Mixins — héritage multiple idiomatique

Un mixin est une classe conçue pour ajouter des fonctionnalités sans être instanciée seule.

```python
class JsonMixin:
    def to_json(self):
        import json
        return json.dumps(vars(self))

class LogMixin:
    def log(self):
        print(f"[LOG] {repr(self)}")

class Utilisateur(JsonMixin, LogMixin):
    def __init__(self, nom, age):
        self.nom = nom
        self.age = age

    def __repr__(self):
        return f"Utilisateur({self.nom})"

u = Utilisateur("Alice", 30)
u.to_json()   # '{"nom": "Alice", "age": 30}'
u.log()       # [LOG] Utilisateur(Alice)
```

**Conventions mixin :**
- Le nom se termine par `Mixin`
- Pas d'état propre (pas de `__init__` ou minimal)
- Pas destinée à être instanciée seule
- Placée en tête de liste des parents : `class Enfant(Mixin1, Mixin2, BaseConcrète)`

## Conflits de noms

```python
class A:
    def methode(self): return "A"

class B:
    def methode(self): return "B"

class C(A, B):
    pass

C().methode()   # "A" — premier parent dans le MRO gagne

class C(B, A):
    pass
C().methode()   # "B"
```

> [!warning] Héritage multiple complexe → difficile à déboguer
> Préférer la composition ou les mixins simples à une hiérarchie profonde avec héritage multiple. Si le MRO surprend, l'afficher avec `Classe.__mro__` pour diagnostiquer.
