#python #oop #attributs #méthodes

## Attributs d'instance

Créés dans `__init__` (ou n'importe quelle méthode) via `self.nom = valeur`.

```python
class Voiture:
    def __init__(self, marque, vitesse_max):
        self.marque = marque
        self.vitesse_max = vitesse_max
        self._vitesse = 0          # convention : _ = usage interne
        self.__kilometrage = 0     # __ = name mangling (voir ci-dessous)
```

## Conventions de nommage

| Convention | Signification |
|-----------|--------------|
| `nom` | Attribut public — accessible librement |
| `_nom` | Usage interne — à ne pas utiliser depuis l'extérieur (convention, pas enforced) |
| `__nom` | Name mangling — renommé en `_Classe__nom` pour éviter les conflits en héritage |

```python
v = Voiture("Toyota", 180)
v.marque           # "Toyota" ✅
v._vitesse         # 0 — accessible mais déconseillé depuis l'extérieur
v.__kilometrage    # AttributeError ❌
v._Voiture__kilometrage  # 0 — accès via le vrai nom (name mangling)
```

## Méthodes d'instance

Reçoivent `self` en premier paramètre — accès à tous les attributs de l'instance.

```python
class Voiture:
    def __init__(self, marque):
        self.marque = marque
        self._vitesse = 0

    def accelerer(self, delta):
        self._vitesse = min(self._vitesse + delta, self.vitesse_max)

    def freiner(self, delta):
        self._vitesse = max(self._vitesse - delta, 0)

    def etat(self):
        return f"{self.marque} à {self._vitesse} km/h"
```

## Méthodes sans état — délégation

```python
class Calculateur:
    def __init__(self, valeur):
        self.valeur = valeur

    def double(self):
        return self.valeur * 2      # utilise self

    def describe(self):
        print(f"Valeur : {self.valeur}, double : {self.double()}")
```

## Attributs dynamiques

Python permet d'ajouter des attributs à n'importe quel moment.

```python
p = Point(1, 2)
p.label = "origin"   # ajouté après création — valide mais peu recommandé
```

Pour interdire les attributs non déclarés, utiliser `__slots__` :

```python
class Point:
    __slots__ = ("x", "y")   # seuls x et y sont autorisés

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
p.z = 3    # AttributeError ❌ — aussi plus performant en mémoire
```

## `__slots__` — quand l'utiliser

| | Sans `__slots__` | Avec `__slots__` |
|---|---|---|
| `__dict__` | ✅ présent | ❌ absent |
| Attributs dynamiques | ✅ | ❌ |
| Mémoire | Plus | Moins |
| Performance | Normale | Légèrement meilleure |

> [!tip] `__slots__` pour les classes à nombreuses instances
> Si vous créez des millions d'instances (ex : nœuds d'un graphe), `__slots__` réduit significativement la consommation mémoire.
