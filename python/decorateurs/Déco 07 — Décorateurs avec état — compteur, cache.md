#python #decorateurs #état #cache #compteur

## État dans la closure
```python
from functools import wraps

def count_calls(func):
    """Compte le nombre d'appels à la fonction."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.call_count += 1        # état stocké sur la fonction wrapper
        return func(*args, **kwargs)
    wrapper.call_count = 0             # initialisation de l'état
    return wrapper

@count_calls
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)
add(3, 4)
add(5, 6)
print(add.call_count)   # 3
```

## État avec un dictionnaire (mutable dans la closure)
```python
def memoize(func):
    """Cache simple sans expiration."""
    cache = {}
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    wrapper.cache = cache         # exposition du cache pour inspection
    wrapper.cache_clear = cache.clear
    return wrapper

@memoize
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(10)          # calcul
fibonacci(10)          # depuis le cache
fibonacci.cache        # {(0,): 0, (1,): 1, (2,): 1, ...}
fibonacci.cache_clear() # vider le cache
```

## Timer — mesurer le temps d'exécution
```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start  = time.perf_counter()
        result = func(*args, **kwargs)
        end    = time.perf_counter()
        print(f"{func.__name__} : {end - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function(n: int) -> int:
    time.sleep(0.1)
    return sum(range(n))

slow_function(1_000_000)   # slow_function : 0.1032s
```

## Rate limiter — limiter la fréquence d'appel
```python
import time
from functools import wraps

def rate_limit(calls_per_second: float):
    min_interval = 1.0 / calls_per_second
    def decorator(func):
        last_called = [0.0]          # liste pour contourner nonlocal
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.perf_counter() - last_called[0]
            wait    = min_interval - elapsed
            if wait > 0:
                time.sleep(wait)
            result = func(*args, **kwargs)
            last_called[0] = time.perf_counter()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_second=2)    # max 2 appels par seconde
def call_api(endpoint: str) -> dict:
    ...
```

> [!tip] Stocker l'état sur wrapper vs dans une liste
> Pour un seul compteur : `wrapper.count = 0` puis `wrapper.count += 1`.
> Pour plusieurs variables mutables : utiliser une liste `[val]` ou un dict dans la closure, ou mieux, une classe — voir [[Déco 08 — Décorateurs basés sur une classe]].
