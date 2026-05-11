#python #bases #modules #imports

## Importer un module

```python
import math
math.sqrt(16)      # 4.0 — accès via le nom du module
math.pi            # 3.14159...

import os
import sys
```

## Importer des noms spécifiques

```python
from math import sqrt, pi
sqrt(16)           # 4.0 — sans préfixe
pi                 # 3.14159...

from math import *   # ❌ importe tout — pollue l'espace de noms
```

## Alias

```python
import numpy as np       # convention quasi-universelle
import pandas as pd
import matplotlib.pyplot as plt

from pathlib import Path as P   # moins courant
```

## Structure d'un module

Tout fichier `.py` est un module importable.

```
projet/
├── main.py
└── utils.py
```

```python
# utils.py
def helper():
    return 42

# main.py
import utils
utils.helper()   # 42

# ou
from utils import helper
helper()         # 42
```

## `__name__` et point d'entrée

```python
# utils.py
def helper():
    return 42

if __name__ == "__main__":
    # Ce bloc ne s'exécute que si le fichier est lancé directement
    # Pas si importé par un autre module
    print(helper())
```

```bash
python utils.py   # exécute le bloc __main__
```

## Packages — dossiers de modules

```
monpackage/
├── __init__.py   # fait du dossier un package (peut être vide)
├── module_a.py
└── module_b.py
```

```python
from monpackage import module_a
from monpackage.module_a import ma_fonction
```

## Bibliothèque standard — modules utiles

| Module | Usage |
|--------|-------|
| `math` | sqrt, pi, floor, ceil, log, sin... |
| `os` | Chemins, variables d'env, process |
| `os.path` | Manipulation de chemins (ou `pathlib`) |
| `pathlib` | Chemins orientés objet (`Path`) |
| `sys` | argv, stdin/stdout, path, exit() |
| `json` | Sérialisation JSON |
| `re` | Expressions régulières |
| `datetime` | Dates et heures |
| `collections` | deque, Counter, defaultdict, OrderedDict |
| `itertools` | chain, product, combinations... |
| `functools` | reduce, lru_cache, partial, wraps |
| `random` | Nombres aléatoires |
| `time` | Mesure du temps, sleep |
| `typing` | Annotations de type |

## Exemples rapides

```python
import json
json.dumps({"a": 1})         # '{"a": 1}'
json.loads('{"a": 1}')       # {"a": 1}

from pathlib import Path
p = Path("dossier/fichier.txt")
p.exists()
p.read_text()
p.parent        # dossier
p.stem          # fichier
p.suffix        # .txt

from collections import Counter
Counter([1, 1, 2, 3, 3, 3])  # Counter({3: 3, 1: 2, 2: 1})

from functools import lru_cache
@lru_cache(maxsize=128)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
```

> [!tip] `pathlib` plutôt que `os.path`
> `pathlib.Path` est l'API moderne pour manipuler les chemins. Plus lisible et orientée objet que les fonctions `os.path`.
