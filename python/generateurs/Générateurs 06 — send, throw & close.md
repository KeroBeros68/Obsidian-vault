#python #generateurs #send #throw #close #coroutines

Les générateurs peuvent recevoir des données depuis l'extérieur — pas seulement en envoyer.

## `send(valeur)` — injecter une valeur

`yield` peut être utilisé comme expression — `send()` lui fournit une valeur.

```python
def accumulateur():
    total = 0
    while True:
        valeur = yield total    # yield retourne total, reçoit la prochaine valeur via send
        total += valeur

gen = accumulateur()
next(gen)        # 0  — amorçage obligatoire (avance jusqu'au premier yield)
gen.send(10)     # 10
gen.send(5)      # 15
gen.send(3)      # 18
```

**Règle d'amorçage :** avant le premier `send(valeur)`, appeler `next(gen)` ou `gen.send(None)` pour avancer jusqu'au premier `yield`.

```python
# Décorateur d'amorçage automatique
from functools import wraps

def amorcer(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        gen = f(*args, **kwargs)
        next(gen)
        return gen
    return wrapper

@amorcer
def accumulateur():
    total = 0
    while True:
        valeur = yield total
        total += valeur

gen = accumulateur()
gen.send(10)   # 10 — pas besoin de next() avant ✅
```

## `throw(exception)` — injecter une exception

```python
def gen():
    try:
        yield 1
        yield 2
    except ValueError as e:
        print(f"exception reçue : {e}")
        yield -1

g = gen()
next(g)              # 1
g.throw(ValueError, "oups")
# "exception reçue : oups"
# -1
```

L'exception est levée **à l'endroit où le générateur est suspendu** (au `yield`).

## `close()` — arrêter proprement

Lève `GeneratorExit` à l'endroit du `yield` courant.

```python
def gen_avec_nettoyage():
    try:
        while True:
            yield
    finally:
        print("nettoyage exécuté")   # garanti même si close() est appelé

g = gen_avec_nettoyage()
next(g)
g.close()   # "nettoyage exécuté"
```

`GeneratorExit` ne doit pas être capturée et ignorée — Python lèvera `RuntimeError` si le générateur continue à `yield` après.

## Cycle de vie complet

```
Créé    →   [next() / send(None)]   →   Suspendu au yield
                                              │
                                    ┌─────────┼─────────┐
                               next()    send(val)    throw(exc) / close()
                                              │
                                         yield suivant
                                              │
                              StopIteration / return → Terminé
```

## Résumé des méthodes

| Méthode | Rôle | Valeur reçue par `yield` |
|---------|------|--------------------------|
| `next(gen)` | Avance au prochain yield | `None` |
| `gen.send(val)` | Avance, injecte `val` | `val` |
| `gen.throw(exc)` | Lève `exc` au yield courant | — (exception) |
| `gen.close()` | Lève `GeneratorExit` | — (fin) |

> [!info] Générateurs et `async/await`
> `send`, `throw`, `close` sont les primitives sur lesquelles repose `asyncio`. Un `async def` est un générateur spécialisé — `await` est `yield from` avec des garanties supplémentaires.
