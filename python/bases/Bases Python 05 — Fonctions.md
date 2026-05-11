#python #bases #fonctions

## Définition et appel

```python
def saluer(nom):
    return f"Bonjour {nom}"

resultat = saluer("Alice")   # "Bonjour Alice"
```

## Valeurs par défaut

```python
def puissance(base, exposant=2):
    return base ** exposant

puissance(3)     # 9  — exposant=2 par défaut
puissance(3, 3)  # 27
```

> [!warning] Valeur par défaut mutable
> ```python
> def ajouter(val, liste=[]):   # ❌ la liste est partagée entre les appels
>     liste.append(val)
>     return liste
>
> def ajouter(val, liste=None): # ✅
>     if liste is None:
>         liste = []
>     liste.append(val)
>     return liste
> ```

## Arguments nommés (keyword arguments)

```python
def creer(nom, age, ville="Paris"):
    ...

creer("Alice", 30)                  # positionnel
creer(nom="Alice", age=30)          # nommé
creer("Alice", ville="Lyon", age=30)  # mixte — nommés dans n'importe quel ordre
```

## `*args` et `**kwargs`

```python
def somme(*args):          # args est un tuple
    return sum(args)

somme(1, 2, 3, 4)          # 10

def afficher(**kwargs):    # kwargs est un dict
    for k, v in kwargs.items():
        print(f"{k} = {v}")

afficher(nom="Alice", age=30)
```

```python
def f(a, b, *, force=False):   # * force les suivants à être nommés
    ...

f(1, 2, force=True)   # ✅
f(1, 2, True)         # ❌ TypeError
```

## Déballage à l'appel

```python
args = [1, 2, 3]
somme(*args)           # équivalent à somme(1, 2, 3)

params = {"nom": "Alice", "age": 30}
creer(**params)        # équivalent à creer(nom="Alice", age=30)
```

## Retour multiple

```python
def minmax(lst):
    return min(lst), max(lst)   # retourne un tuple

petit, grand = minmax([3, 1, 4, 1, 5])  # déballage ✅
```

## Portée — LEGB

Python cherche une variable dans l'ordre : **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in.

```python
x = "global"

def f():
    x = "local"     # nouvelle variable locale — ne modifie pas le global
    print(x)        # "local"

def g():
    global x        # déclare qu'on veut le x global
    x = "modifié"

def externe():
    y = "enclosing"
    def interne():
        nonlocal y  # accède au y de externe
        y = "modifié"
    interne()
    print(y)        # "modifié"
```

## Fonctions de première classe

```python
def appliquer(fn, valeur):
    return fn(valeur)

appliquer(str.upper, "hello")   # "HELLO"
appliquer(len, [1, 2, 3])       # 3

fonctions = [abs, str, bool]
fonctions[0](-5)   # 5
```

## Lambda

```python
carre = lambda x: x ** 2
carre(4)   # 16

# Utile comme argument de tri ou de transformation
noms = ["Charlie", "Alice", "Bob"]
sorted(noms, key=lambda n: len(n))   # ["Bob", "Alice", "Charlie"]
```

> [!tip] `lambda` pour les cas simples uniquement
> Si la logique dépasse une expression, définir une vraie fonction avec `def` — c'est plus lisible et débogable.
