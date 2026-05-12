#python #generateurs #yield-from #délégation

## `yield from` — déléguer à un sous-itérable

```python
def gen_a():
    yield 1
    yield 2

def gen_b():
    yield 3
    yield 4

def combiné():
    yield from gen_a()   # délègue complètement à gen_a
    yield from gen_b()   # puis à gen_b

list(combiné())   # [1, 2, 3, 4]
```

`yield from iterable` est équivalent à `for x in iterable: yield x`, mais avec des différences importantes pour les sous-générateurs.

## Équivalence et différence

```python
# Équivalent basique
def sans_yield_from(gen):
    for val in gen:
        yield val

# Avec yield from — plus court et gère aussi send/throw/return
def avec_yield_from(gen):
    yield from gen
```

La différence est visible dès qu'on utilise `send()`, `throw()` ou la valeur de `return` du sous-générateur (fiche 06).

## Aplatir des itérables imbriqués

```python
def aplatir(iterable):
    for element in iterable:
        if hasattr(element, '__iter__') and not isinstance(element, str):
            yield from aplatir(element)   # récursion sur les sous-itérables
        else:
            yield element

list(aplatir([1, [2, [3, 4]], 5, [6]]))
# [1, 2, 3, 4, 5, 6]
```

## Récupérer la valeur de `return` du sous-générateur

```python
def sous_gen():
    yield 1
    yield 2
    return "résultat du sous-générateur"

def gen_parent():
    valeur_retour = yield from sous_gen()   # capture le return
    print(f"sous-générateur a retourné : {valeur_retour}")
    yield 3

list(gen_parent())
# sous-générateur a retourné : résultat du sous-générateur
# [1, 2, 3]
```

## Déléguer à des séquences simples

`yield from` fonctionne avec n'importe quel itérable, pas seulement les générateurs :

```python
def gen():
    yield from [1, 2, 3]       # liste
    yield from range(4, 7)     # range
    yield from "abc"           # chaîne — caractère par caractère

list(gen())   # [1, 2, 3, 4, 5, 6, 'a', 'b', 'c']
```

## Cas d'usage typique — composition de générateurs

```python
def lire_fichiers(chemins):
    for chemin in chemins:
        yield from open(chemin)   # délègue la ligne à chaque fichier

def filtrer(lignes, motif):
    yield from (l for l in lignes if motif in l)

def traiter(chemins, motif):
    yield from filtrer(lire_fichiers(chemins), motif)
```

> [!tip] `yield from` pour les générateurs récursifs
> Sans `yield from`, un générateur récursif doit faire `for x in appel_recursif(): yield x`. Avec `yield from appel_recursif()`, la délégation est totale et la valeur de `return` est propagée.

> [!info] PEP 380
> `yield from` a été introduit en Python 3.3 (PEP 380). C'est aussi la base des coroutines `async/await` — `await` est une évolution de `yield from`.
