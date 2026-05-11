#python #oop #classmethod #staticmethod

## Les trois types de méthodes

```python
class MaClasse:
    attribut = "classe"

    def methode_instance(self):       # reçoit l'instance
        return self.attribut

    @classmethod
    def methode_classe(cls):          # reçoit la classe
        return cls.attribut

    @staticmethod
    def methode_statique():           # ne reçoit rien
        return "indépendante"
```

## `@classmethod`

Reçoit `cls` (la classe elle-même) au lieu de `self`. Peut accéder et modifier les attributs de classe.

**Cas d'usage principal : constructeurs alternatifs (factory methods)**

```python
class Date:
    def __init__(self, annee, mois, jour):
        self.annee = annee
        self.mois = mois
        self.jour = jour

    @classmethod
    def depuis_chaine(cls, chaine):       # "2024-05-12"
        annee, mois, jour = map(int, chaine.split("-"))
        return cls(annee, mois, jour)     # cls = Date ou sous-classe ✅

    @classmethod
    def aujourdhui(cls):
        from datetime import date
        d = date.today()
        return cls(d.year, d.month, d.day)

d1 = Date(2024, 5, 12)
d2 = Date.depuis_chaine("2024-05-12")
d3 = Date.aujourdhui()
```

```python
# @classmethod et héritage : cls = la sous-classe appelante
class DateFR(Date):
    pass

d = DateFR.depuis_chaine("2024-05-12")
type(d)   # DateFR — cls a bien pris la valeur DateFR ✅
```

## `@staticmethod`

Pas de `self` ni de `cls`. Fonction ordinaire logiquement liée à la classe, sans accès à son état.

```python
class Convertisseur:
    @staticmethod
    def celsius_vers_fahrenheit(c):
        return c * 9 / 5 + 32

    @staticmethod
    def kg_vers_livres(kg):
        return kg * 2.20462

Convertisseur.celsius_vers_fahrenheit(100)   # 212.0
# Peut aussi être appelée sur une instance (mais peu idiomatique)
c = Convertisseur()
c.celsius_vers_fahrenheit(0)   # 32.0
```

## Comparaison

| | `self` | `cls` | Accès instance | Accès classe |
|---|---|---|---|---|
| Méthode d'instance | ✅ | — | ✅ | ✅ |
| `@classmethod` | — | ✅ | ❌ | ✅ |
| `@staticmethod` | — | — | ❌ | ❌ |

## Quand utiliser quoi

```python
class Utilisateur:
    _instances = []

    def __init__(self, nom):
        self.nom = nom
        Utilisateur._instances.append(self)

    # @classmethod — factory ou opération sur la classe
    @classmethod
    def depuis_dict(cls, data):
        return cls(data["nom"])

    @classmethod
    def compter(cls):
        return len(cls._instances)

    # @staticmethod — utilitaire lié au domaine, sans état
    @staticmethod
    def valider_nom(nom):
        return isinstance(nom, str) and len(nom) >= 2
```

> [!tip] `@classmethod` pour les factories, `@staticmethod` pour les helpers
> Si la méthode a besoin de créer une instance ou de connaître la classe (utile en héritage), utiliser `@classmethod`. Si c'est une fonction utilitaire pure sans lien avec l'état, utiliser `@staticmethod` — ou carrément une fonction module-level.
