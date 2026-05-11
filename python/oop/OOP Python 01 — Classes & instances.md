#python #oop #classes #instances

## Définir une classe

```python
class Chien:
    def __init__(self, nom, race):
        self.nom = nom    # attribut d'instance
        self.race = race

    def aboyer(self):
        return f"{self.nom} dit : Ouaf !"
```

`__init__` est appelé automatiquement à la création. `self` désigne l'instance en cours — il est toujours le premier paramètre de toute méthode d'instance.

## Créer des instances

```python
rex = Chien("Rex", "Berger")
fido = Chien("Fido", "Labrador")

rex.nom          # "Rex"
rex.aboyer()     # "Rex dit : Ouaf !"
fido.nom         # "Fido" — chaque instance a ses propres attributs
```

## Attributs de classe vs d'instance

```python
class Compteur:
    total = 0                 # attribut de classe — partagé entre toutes les instances

    def __init__(self, nom):
        Compteur.total += 1
        self.nom = nom        # attribut d'instance — propre à chaque objet
        self.id = Compteur.total

a = Compteur("a")
b = Compteur("b")
Compteur.total    # 2
a.total           # 2 — accès possible via l'instance (cherche en classe si absent)
a.id              # 1
b.id              # 2
```

> [!warning] Attribut de classe mutable
> ```python
> class A:
>     items = []           # partagé entre toutes les instances
>
> a = A()
> a.items.append(1)        # modifie la liste de la CLASSE ❌
>
> class A:
>     def __init__(self):
>         self.items = []  # ✅ chaque instance a sa propre liste
> ```

## `__repr__` et `__str__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x}, {self.y})"   # repr — pour les développeurs

    def __str__(self):
        return f"({self.x}, {self.y})"        # str — pour l'affichage

p = Point(3, 4)
repr(p)    # "Point(3, 4)"
str(p)     # "(3, 4)"
print(p)   # "(3, 4)"  — print appelle __str__
```

> [!tip] Toujours définir `__repr__`
> Si `__str__` est absent, Python utilise `__repr__`. Définir au minimum `__repr__` pour que l'objet s'affiche de façon utile dans le REPL et les logs.

## Inspecter une instance

```python
vars(p)         # {'x': 3, 'y': 4} — dict des attributs d'instance
p.__dict__      # identique
dir(p)          # liste tous les attributs et méthodes disponibles

hasattr(p, 'x')        # True
getattr(p, 'x')        # 3
getattr(p, 'z', 0)     # 0 — valeur par défaut si absent
setattr(p, 'z', 99)    # p.z = 99
delattr(p, 'z')        # del p.z
```
