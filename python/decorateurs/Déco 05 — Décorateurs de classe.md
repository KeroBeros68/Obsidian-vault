#python #decorateurs #classes #méthodes

## Décorateurs sur les méthodes d'une classe
```python
class MyClass:
    @staticmethod
    def static_method(): ...          # pas d'accès à self ni cls

    @classmethod
    def class_method(cls): ...        # accès à la classe via cls

    @property
    def value(self): ...              # getter — accès comme un attribut

    @value.setter
    def value(self, v): ...           # setter

    @value.deleter
    def value(self): ...              # deleter
```

## @property — attribut calculé
```python
class Circle:
    def __init__(self, radius: float) -> None:
        self._radius = radius

    @property
    def radius(self) -> float:
        return self._radius

    @radius.setter
    def radius(self, value: float) -> None:
        if value < 0:
            raise ValueError("Le rayon doit être positif")
        self._radius = value

    @property
    def area(self) -> float:              # propriété en lecture seule
        return 3.14159 * self._radius ** 2

c = Circle(5)
print(c.radius)   # 5.0
print(c.area)     # 78.53...
c.radius = 10     # ✅ appelle le setter
c.area = 100      # ❌ AttributeError : pas de setter
```

## @staticmethod vs @classmethod
```python
class Counter:
    _count: int = 0

    @classmethod
    def increment(cls) -> None:
        cls._count += 1       # accès à la classe — héritage compatible

    @classmethod
    def get_count(cls) -> int:
        return cls._count

    @staticmethod
    def validate(value: int) -> bool:
        return value >= 0     # pas besoin de cls ni self — simple utilitaire

Counter.increment()
Counter.get_count()      # 1
Counter.validate(-1)     # False
```

## Décorateur appliqué à une classe entière
```python
from functools import wraps

def singleton(cls):
    """Transforme une classe en singleton."""
    instances = {}
    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class Database:
    def __init__(self, url: str) -> None:
        self.url = url
        print(f"Connexion à {url}")

db1 = Database("postgresql://localhost/dev")   # "Connexion à postgresql://localhost/dev"
db2 = Database("postgresql://localhost/dev")   # rien — retourne la même instance
db1 is db2   # True ✅
```

## Décorateur de classe — ajouter des méthodes dynamiquement
```python
def add_repr(cls):
    """Ajoute un __repr__ automatique à la classe."""
    def __repr__(self) -> str:
        attrs = ", ".join(
            f"{k}={v!r}"
            for k, v in self.__dict__.items()
            if not k.startswith("_")
        )
        return f"{cls.__name__}({attrs})"
    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

Point(1.0, 2.0)   # Point(x=1.0, y=2.0)
```

> [!tip] @property — convention de nommage
> L'attribut interne prend un underscore : `self._radius`.
> La property publique expose `radius` (sans underscore).
> Cela évite les récursions infinies dans le setter.
