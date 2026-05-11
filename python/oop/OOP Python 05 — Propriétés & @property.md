#python #oop #property #accesseurs

## Le problème sans `@property`

```python
class Cercle:
    def __init__(self, rayon):
        self.rayon = rayon   # attribut public — pas de contrôle

c = Cercle(5)
c.rayon = -3    # ❌ valeur invalide acceptée silencieusement
```

## `@property` — getter

Transforme une méthode en attribut en lecture seule.

```python
import math

class Cercle:
    def __init__(self, rayon):
        self._rayon = rayon

    @property
    def rayon(self):
        return self._rayon

    @property
    def aire(self):
        return math.pi * self._rayon ** 2   # calculé à la demande

c = Cercle(5)
c.rayon      # 5  — appelle le getter, syntaxe d'attribut
c.aire       # 78.53...
c.rayon = 3  # AttributeError — pas de setter défini
```

## `@x.setter` — setter avec validation

```python
class Cercle:
    def __init__(self, rayon):
        self.rayon = rayon       # passe par le setter ✅

    @property
    def rayon(self):
        return self._rayon

    @rayon.setter
    def rayon(self, valeur):
        if valeur < 0:
            raise ValueError(f"rayon ne peut pas être négatif : {valeur}")
        self._rayon = valeur

c = Cercle(5)
c.rayon = 3     # ✅
c.rayon = -1    # ValueError ❌
```

## `@x.deleter`

```python
class Cache:
    def __init__(self):
        self._data = {}

    @property
    def data(self):
        return self._data

    @data.deleter
    def data(self):
        self._data.clear()
        print("cache vidé")

c = Cache()
del c.data   # appelle le deleter
```

## Propriété calculée (computed property)

```python
class Rectangle:
    def __init__(self, largeur, hauteur):
        self.largeur = largeur
        self.hauteur = hauteur

    @property
    def surface(self):
        return self.largeur * self.hauteur   # recalculé à chaque accès

    @property
    def perimetre(self):
        return 2 * (self.largeur + self.hauteur)

r = Rectangle(4, 5)
r.surface     # 20
r.largeur = 6
r.surface     # 30 — mise à jour automatique
```

## Comparaison attribut public vs `@property`

| | Attribut public | `@property` |
|---|---|---|
| Syntaxe d'accès | `obj.x` | `obj.x` |
| Validation | ❌ | ✅ |
| Valeur calculée | ❌ | ✅ |
| Compatibilité API | — | Transparente |

> [!tip] Commencer par un attribut public
> Ne pas créer une property par défaut. Si la validation ou le calcul devient nécessaire, la migration vers `@property` ne casse pas l'API (l'accès reste `obj.x`).
