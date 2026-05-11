#python #oop #abc #interfaces #duck-typing #dataclasses

## Duck typing

Python ne vérifie pas le type d'un objet — seulement s'il a les méthodes et attributs nécessaires.

```python
class Chien:
    def parler(self): return "Ouaf"

class Chat:
    def parler(self): return "Miaou"

class Robot:
    def parler(self): return "Bip bop"

def faire_parler(animal):
    print(animal.parler())   # fonctionne avec n'importe quel objet qui a .parler()

faire_parler(Chien())   # "Ouaf"
faire_parler(Chat())    # "Miaou"
faire_parler(Robot())   # "Bip bop" ✅ — pas de lien d'héritage requis
```

> *"If it walks like a duck and quacks like a duck, it's a duck."*

## Classes abstraites — `abc.ABC`

Garantissent qu'une sous-classe implémente certaines méthodes avant d'être instanciable.

```python
from abc import ABC, abstractmethod

class Forme(ABC):
    @abstractmethod
    def aire(self) -> float:
        ...

    @abstractmethod
    def perimetre(self) -> float:
        ...

    def describe(self):          # méthode concrète — héritée telle quelle
        return f"Aire={self.aire():.2f}, Périmètre={self.perimetre():.2f}"

class Cercle(Forme):
    def __init__(self, r):
        self.r = r

    def aire(self):
        import math
        return math.pi * self.r ** 2

    def perimetre(self):
        import math
        return 2 * math.pi * self.r

Forme()       # TypeError — ne peut pas instancier une classe abstraite ❌
Cercle(5)     # ✅ — toutes les méthodes abstraites sont implémentées
```

```python
class CarreSansAire(Forme):
    def perimetre(self): return 4

CarreSansAire()   # TypeError — aire() non implémentée ❌
```

## `@abstractmethod` + `@property`

Forcer les sous-classes à implémenter une propriété — en lecture seule ou lecture/écriture.

**Getter abstrait seul (lecture seule) :**

```python
class Vehicule(ABC):
    @property
    @abstractmethod
    def vitesse_max(self) -> float:
        ...

class Voiture(Vehicule):
    @property
    def vitesse_max(self) -> float:
        return 180.0
```

**Getter + Setter abstraits (lecture/écriture) :**

```python
class INode(ABC):
    @property
    @abstractmethod
    def max_drones(self) -> int:
        ...

    @max_drones.setter
    @abstractmethod
    def max_drones(self, value: int) -> None:
        ...
```

La sous-classe concrète doit redéfinir **les deux** avec `@property` puis `@x.setter` :

```python
class DroneNode(INode):
    def __init__(self, capacite: int):
        self._max_drones = capacite

    @property
    def max_drones(self) -> int:
        return self._max_drones

    @max_drones.setter
    def max_drones(self, value: int) -> None:
        if value < 0:
            raise ValueError("capacité négative")
        self._max_drones = value
```

> [!warning] Ordre des décorateurs — `@property` avant `@abstractmethod`
> ```python
> # ✅ correct
> @property
> @abstractmethod
> def x(self): ...
>
> # ❌ incorrect — abstractmethod n'encapsule pas une property
> @abstractmethod
> @property
> def x(self): ...
> ```
> `@property` doit être le décorateur le plus externe (premier lu, dernier appliqué).

> [!warning] Oublier le setter dans la sous-classe
> Si la sous-classe n'implémente que le getter d'une propriété abstraite getter+setter, Python **lève quand même `TypeError` à l'instanciation** car le setter abstrait n'est pas satisfait.

## `Protocol` — duck typing structurel formel

Pour vérifier le duck typing à la compilation (mypy/pyright) sans héritage.

```python
from typing import Protocol

class Parlant(Protocol):
    def parler(self) -> str: ...

def faire_parler(animal: Parlant) -> None:
    print(animal.parler())

# Chien n'hérite PAS de Parlant — mais mypy est satisfait si parler() existe
class Chien:
    def parler(self) -> str:
        return "Ouaf"

faire_parler(Chien())   # ✅
```

→ Voir [[Typing 06 — Protocoles & duck typing structurel]] pour le détail.

## `dataclasses` — classes de données sans boilerplate

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    label: str = "point"       # valeur par défaut

@dataclass
class Polygone:
    sommets: list = field(default_factory=list)  # mutable : factory ✅

p = Point(1.0, 2.0)
p.x          # 1.0
repr(p)      # "Point(x=1.0, y=2.0, label='point')" — __repr__ généré
p == Point(1.0, 2.0)   # True — __eq__ généré
```

```python
@dataclass(frozen=True)   # immuable — génère __hash__
class Coordonnee:
    lat: float
    lon: float
```

## Quand utiliser quoi

| Besoin | Solution |
|--------|---------|
| Typage structurel sans héritage | Duck typing |
| Contrat formel vérifié à l'exécution | `ABC` + `@abstractmethod` |
| Contrat vérifié statiquement (mypy) | `Protocol` |
| Classe de données sans logique | `@dataclass` |
| Classe de données immuable | `@dataclass(frozen=True)` |
