#python #tqdm #itérables #boucles #progression

## Cas d'usage de base
```python
from tqdm import tqdm
import time

# Liste
for item in tqdm([1, 2, 3, 4, 5]):
    time.sleep(0.5)

# range
for i in tqdm(range(1000)):
    pass

# Générateur — fournir `total` si la longueur n'est pas connue
def gen():
    for i in range(100):
        yield i

for item in tqdm(gen(), total=100):
    pass

# Fichier ligne par ligne
with open("data.txt") as f:
    for line in tqdm(f, total=10_000, desc="Lecture"):
        process(line)
```

## Avec description et unité
```python
from tqdm import tqdm

files = ["a.json", "b.json", "c.json"]

for f in tqdm(files, desc="Traitement fichiers", unit="fichier"):
    process(f)
# Traitement fichiers:  67%|██████████▋     | 2/3 [00:01<00:00, 2.00 fichier/s]
```

## trange — raccourci pour range
```python
from tqdm import trange

for i in trange(1000, desc="Calcul"):
    pass
# Équivalent à : for i in tqdm(range(1000), desc="Calcul"):
```

## Avec enumerate
```python
from tqdm import tqdm

items = ["a", "b", "c", "d"]

for i, item in enumerate(tqdm(items)):
    print(i, item)

# Ou avec tqdm.auto pour accéder à l'index via le contexte
for i, item in tqdm(enumerate(items), total=len(items)):
    print(i, item)
```

## Avec zip et map
```python
from tqdm import tqdm

keys   = range(100)
values = range(100, 200)

for k, v in tqdm(zip(keys, values), total=100):
    pass

# map + tqdm
results = list(tqdm(map(process, items), total=len(items)))
```

## Concaténer plusieurs itérables
```python
from tqdm import tqdm
import itertools

lists = [[1,2,3], [4,5,6], [7,8,9]]
total = sum(len(l) for l in lists)

for item in tqdm(itertools.chain(*lists), total=total, desc="Chaîne"):
    pass
```

> [!tip] `total` est essentiel pour les générateurs
> Sans `total`, tqdm ne peut pas afficher le pourcentage ni le temps restant — seulement un compteur défilant. Toujours passer `total=len(data)` ou le nombre d'items connu.
