#python #oop #héritage #super

## Hériter d'une classe

```python
class Animal:
    def __init__(self, nom):
        self.nom = nom

    def parler(self):
        return "..."

class Chien(Animal):          # Chien hérite de Animal
    def parler(self):         # surcharge (override)
        return f"{self.nom} dit : Ouaf !"

class Chat(Animal):
    def parler(self):
        return f"{self.nom} dit : Miaou !"

rex = Chien("Rex")
rex.parler()    # "Rex dit : Ouaf !"
rex.nom         # "Rex" — hérité de Animal
```

## `super()` — appeler le parent

```python
class Animal:
    def __init__(self, nom, age):
        self.nom = nom
        self.age = age

class Chien(Animal):
    def __init__(self, nom, age, race):
        super().__init__(nom, age)   # délègue au parent ✅
        self.race = race             # attribut propre à Chien

rex = Chien("Rex", 3, "Berger")
rex.nom    # "Rex"
rex.race   # "Berger"
```

```python
class Chien(Animal):
    def parler(self):
        base = super().parler()      # appel de la méthode parente
        return f"{base} (et remue la queue)"
```

## Vérifier le type

```python
rex = Chien("Rex", 3, "Berger")

isinstance(rex, Chien)    # True
isinstance(rex, Animal)   # True — une instance est aussi du type parent
isinstance(rex, Chat)     # False

issubclass(Chien, Animal) # True
issubclass(Animal, Chien) # False
type(rex) == Chien        # True — type exact uniquement, sans héritage
```

> [!tip] `isinstance` plutôt que `type() ==`
> `isinstance` respecte la hiérarchie d'héritage et les classes abstraites. Préférer `isinstance` dans presque tous les cas.

## Surcharger vs étendre

```python
class Base:
    def traiter(self):
        print("étape 1")
        print("étape 2")

# Remplacement total
class Enfant(Base):
    def traiter(self):
        print("traitement différent")   # Base.traiter() n'est plus appelé

# Extension
class Enfant(Base):
    def traiter(self):
        super().traiter()               # étape 1 + étape 2
        print("étape 3 supplémentaire") # ajout
```

## Héritage et attributs

```python
class Forme:
    couleur = "rouge"         # attribut de classe hérité

class Cercle(Forme):
    pass

Cercle.couleur    # "rouge" — hérité
c = Cercle()
c.couleur         # "rouge"

Cercle.couleur = "bleu"    # modifie uniquement la classe Cercle
Forme.couleur              # "rouge" — inchangé
```

## Classe de base `object`

Toute classe Python hérite implicitement de `object`.

```python
class MaClasse:     # équivalent à class MaClasse(object):
    pass

isinstance(MaClasse(), object)   # True
```

`object` fournit les méthodes par défaut : `__repr__`, `__eq__`, `__hash__`, `__init__`...
