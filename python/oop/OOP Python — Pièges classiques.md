#python #oop #pièges #erreurs #debugging

## 🪤 Piège 1 — Attribut de classe mutable partagé

```python
class Equipe:
    membres = []    # ❌ partagé entre toutes les instances

e1 = Equipe()
e2 = Equipe()
e1.membres.append("Alice")
e2.membres   # ["Alice"] — e2 aussi modifié !

class Equipe:
    def __init__(self):
        self.membres = []    # ✅ chaque instance a sa propre liste
```

---

## 🪤 Piège 2 — Oublier `self` dans une méthode

```python
class Compteur:
    def __init__(self):
        self.val = 0

    def incrementer():    # ❌ pas de self
        self.val += 1

c = Compteur()
c.incrementer()   # TypeError: incrementer() takes 0 positional arguments but 1 was given

    def incrementer(self):   # ✅
        self.val += 1
```

---

## 🪤 Piège 3 — `__init__` parent non appelé

```python
class Animal:
    def __init__(self, nom):
        self.nom = nom

class Chien(Animal):
    def __init__(self, nom, race):
        self.race = race   # ❌ oubli de super().__init__
        # self.nom n'est jamais créé

class Chien(Animal):
    def __init__(self, nom, race):
        super().__init__(nom)   # ✅
        self.race = race
```

---

## 🪤 Piège 4 — Modifier `__dict__` au lieu de l'attribut

```python
p = Point(1, 2)
p.__dict__["x"] = 99   # ❌ contourne @property, validations, __slots__
p.x = 99               # ✅ passe par les mécanismes normaux
```

---

## 🪤 Piège 5 — `__eq__` sans `__hash__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    # ❌ Python met automatiquement __hash__ à None — l'objet ne peut plus être clé de dict

# Fixer __hash__ explicitement si l'objet doit rester hashable
    def __hash__(self):
        return hash((self.x, self.y))   # ✅
```

---

## 🪤 Piège 6 — `is` au lieu de `isinstance` pour tester le type

```python
def traiter(forme):
    if type(forme) == Cercle:   # ❌ exclut les sous-classes de Cercle
        ...
    if isinstance(forme, Cercle):   # ✅ inclut les sous-classes
        ...
```

---

## 🪤 Piège 7 — `super()` oublié dans un mixin avec héritage multiple

```python
class A:
    def f(self): print("A")

class MixinB:
    def f(self):
        print("B")
        # super().f() oublié ❌ — A.f() ne sera jamais appelé

class C(MixinB, A):
    pass

C().f()   # "B" seulement — A ignoré
```

Chaque classe dans une chaîne de mixins doit appeler `super()` pour que le MRO soit respecté.

---

## 🪤 Piège 8 — Propriété sans setter → erreur silencieuse

```python
class Config:
    @property
    def debug(self):
        return self._debug

c = Config()
c.debug = True   # AttributeError: can't set attribute ❌
                 # Si oublié, le message est peu clair
```

Définir le setter si l'attribut doit être modifiable, ou documenter qu'il est en lecture seule.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Attribut de classe mutable | Initialiser dans `__init__` |
| Méthode sans `self` | Toujours déclarer `self` en premier |
| `super().__init__` oublié | Appeler systématiquement en sous-classe |
| `__eq__` sans `__hash__` | Définir `__hash__` ou utiliser `@dataclass(frozen=True)` |
| `type()` au lieu de `isinstance` | Préférer `isinstance` |
| `super()` absent dans un mixin | Appeler `super()` dans toute méthode de mixin |
