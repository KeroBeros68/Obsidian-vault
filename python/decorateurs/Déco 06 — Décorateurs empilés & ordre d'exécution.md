#python #decorateurs #empilement #ordre

## Empiler plusieurs décorateurs
```python
from functools import wraps

def bold(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

def uppercase(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs).upper()
    return wrapper

@bold
@italic
@uppercase
def greet(name: str) -> str:
    return f"bonjour {name}"

greet("Alice")   # "<b><i>BONJOUR ALICE</i></b>"
```

## Ordre d'application — de bas en haut
```python
# @bold
# @italic
# @uppercase
# def greet(): ...

# Équivalent à :
greet = bold(italic(uppercase(greet)))
#            ↑ appliqué en premier (plus proche de la fonction)
#       ↑ appliqué en deuxième
# ↑ appliqué en dernier (plus éloigné)
```

## Ordre d'exécution — de haut en bas
```python
def deco_a(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("A avant")
        result = func(*args, **kwargs)
        print("A après")
        return result
    return wrapper

def deco_b(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("B avant")
        result = func(*args, **kwargs)
        print("B après")
        return result
    return wrapper

@deco_a
@deco_b
def hello():
    print("hello")

hello()
# A avant     ← deco_a s'exécute en premier (le plus haut)
# B avant     ← deco_b s'exécute en deuxième
# hello
# B après     ← deco_b se termine en premier
# A après     ← deco_a se termine en dernier
```

## Visualisation — pile d'appels
```
Application (bas → haut) :  uppercase → italic → bold
Exécution   (haut → bas) :  bold → italic → uppercase → func → uppercase → italic → bold
```

## Décorateurs et méthodes — ordre important
```python
class MyClass:
    @classmethod           # ← doit être en dernier (le plus haut)
    @my_decorator          # ← appliqué en premier à la fonction
    def my_method(cls): ...

    @staticmethod          # ← doit être en dernier
    @my_decorator
    def other_method(): ...
```

> [!warning] @classmethod et @staticmethod doivent être en dernier (le plus haut)
> Ces décorateurs de la librairie standard transforment l'objet descripteur.
> Les autres décorateurs doivent être appliqués avant (plus bas), sinon ils reçoivent un descripteur, pas une fonction.
