#python #decorateurs #fonctions #closures #bases

## Les fonctions sont des objets de première classe
```python
# Une fonction peut être assignée à une variable
def greet(name: str) -> str:
    return f"Bonjour {name}"

say_hello = greet            # pas d'appel — on copie la référence
say_hello("Alice")           # "Bonjour Alice"

# Passée en argument
def apply(func, value):
    return func(value)

apply(greet, "Bob")          # "Bonjour Bob"
apply(str.upper, "hello")    # "HELLO"

# Retournée par une fonction
def get_greeter():
    return greet

greeter = get_greeter()
greeter("Charlie")           # "Bonjour Charlie"
```

## Fonctions définies à l'intérieur d'autres fonctions
```python
def outer():
    def inner():           # inner n'existe que dans outer
        return "inner"
    return inner()

outer()   # "inner"
inner()   # ❌ NameError — inner n'existe pas ici
```

## Closures — capturer le contexte
```python
def make_multiplier(factor: int):
    def multiply(x: int) -> int:
        return x * factor   # factor est capturé depuis l'environnement extérieur
    return multiply         # on retourne la fonction, pas le résultat

double = make_multiplier(2)
triple = make_multiplier(3)

double(5)   # 10
triple(5)   # 15

# Inspecter la closure
double.__closure__          # (<cell at 0x...>,)
double.__closure__[0].cell_contents   # 2
```

## Closures et variables mutables
```python
# Problème classique — variable capturée par référence
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]   # [2, 2, 2] ❌ — i vaut 2 à la fin

# ✅ Capturer par valeur avec un argument par défaut
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]   # [0, 1, 2] ✅

# ✅ Ou avec une closure explicite
def make_fn(i: int):
    def fn(): return i
    return fn

funcs = [make_fn(i) for i in range(3)]
[f() for f in funcs]   # [0, 1, 2] ✅
```

## nonlocal — modifier une variable de la closure
```python
def make_counter():
    count = 0
    def increment():
        nonlocal count      # sans nonlocal → UnboundLocalError
        count += 1
        return count
    return increment

counter = make_counter()
counter()   # 1
counter()   # 2
counter()   # 3
```

> [!tip] Les closures sont la base des décorateurs
> Un décorateur est une fonction qui retourne une autre fonction (wrapper) qui capture la fonction originale dans sa closure. Tout le reste découle de ce principe.
