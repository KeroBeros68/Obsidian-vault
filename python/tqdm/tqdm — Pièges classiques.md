#python #tqdm #pièges #erreurs #debugging

## Piège 1 — Barre sans `total` sur un générateur
```python
from tqdm import tqdm

def gen():
    for i in range(100):
        yield i

# ❌ Pas de total → pas de %, pas de temps restant
for item in tqdm(gen()):
    pass
# 87it [00:04, 19.97it/s]  ← juste un compteur défilant

# ✅ Toujours passer total pour les générateurs
for item in tqdm(gen(), total=100):
    pass
```

## Piège 2 — `print()` casse l'affichage de la barre
```python
from tqdm import tqdm

# ❌ print() écrit directement dans stdout → désaligne la barre
for i in tqdm(range(100)):
    if i % 10 == 0:
        print(f"Checkpoint {i}")   # ❌

# ✅ Utiliser tqdm.write()
for i in tqdm(range(100)):
    if i % 10 == 0:
        tqdm.write(f"Checkpoint {i}")   # ✅
```

## Piège 3 — Barre non fermée en cas d'exception
```python
from tqdm import tqdm

# ❌ Si une exception est levée, pbar n'est pas fermée → terminal corrompu
pbar = tqdm(total=100)
pbar.update(50)
raise ValueError("Oups")   # pbar.close() jamais appelé !

# ✅ Context manager — fermeture garantie
with tqdm(total=100) as pbar:
    pbar.update(50)
    raise ValueError("Oups")   # pbar.close() appelé dans __exit__
```

## Piège 4 — Barres imbriquées sans `leave=False` sur la barre interne
```python
from tqdm import tqdm

# ❌ Chaque barre interne reste affichée → accumulation de lignes
for epoch in tqdm(range(5), desc="Epochs"):
    for batch in tqdm(range(20), desc="Batches"):   # leave=True par défaut !
        pass

# ✅ leave=False sur la barre interne
for epoch in tqdm(range(5), desc="Epochs", position=0, leave=True):
    for batch in tqdm(range(20), desc="Batches", position=1, leave=False):
        pass
```

## Piège 5 — `tqdm.pandas()` non appelé
```python
import pandas as pd
from tqdm import tqdm

df = pd.DataFrame({"val": range(1000)})

# ❌ AttributeError : 'Series' has no attribute 'progress_apply'
df["val"].progress_apply(lambda x: x * 2)

# ✅ Appeler tqdm.pandas() UNE fois avant
tqdm.pandas(desc="Pandas")
df["val"].progress_apply(lambda x: x * 2)
```

## Piège 6 — Barre dans un test ou CI
```python
from tqdm import tqdm

# ❌ Tests lents + logs pollués
def test_my_function():
    for i in tqdm(range(1000)):
        my_function(i)

# ✅ disable=True dans les tests
def test_my_function():
    for i in tqdm(range(1000), disable=True):
        my_function(i)
```

## Piège 7 — update() après close()
```python
from tqdm import tqdm

# ❌ Appeler update() après close() → comportement indéfini
pbar = tqdm(total=100)
pbar.update(50)
pbar.close()
pbar.update(10)   # ❌

# ✅ Toujours utiliser le context manager
with tqdm(total=100) as pbar:
    pbar.update(50)
```

## Piège 8 — Overhead sur des millions de petits items
```python
from tqdm import tqdm

# ❌ Rafraîchissements constants → overhead mesurable
for i in tqdm(range(10_000_000)):
    pass

# ✅ Réduire la fréquence de mise à jour
for i in tqdm(range(10_000_000), miniters=10_000, mininterval=0.5):
    pass
```

## Piège 9 — Mauvais import pour Jupyter
```python
# ❌ Barre ASCII au lieu du widget
from tqdm import tqdm

# ✅ Auto-sélectionne le widget dans Jupyter
from tqdm.auto import tqdm
```

## Piège 10 — Postfix avec objet NumPy non formaté
```python
from tqdm import tqdm
import numpy as np

# ❌ Affichage verbeux : "loss=0.23409999..."
pbar.set_postfix({"loss": np.float32(0.2341)})

# ✅ Formater explicitement
pbar.set_postfix({"loss": f"{float(loss):.4f}"})
```

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Générateur sans `total` | Passer `total=len(data)` |
| `print()` casse la barre | Utiliser `tqdm.write()` |
| Barre non fermée | Context manager `with tqdm(...) as pbar` |
| Barres imbriquées résiduelles | `leave=False` sur la barre interne + `position` |
| `progress_apply` sans init | Appeler `tqdm.pandas()` une fois avant |
| Barre dans les tests / CI | `disable=True` |
| Overhead sur millions d'items | `miniters=N` et `mininterval=0.5` |
| Mauvais import dans Jupyter | `from tqdm.auto import tqdm` |
| Postfix non formaté | `f"{val:.4f}"` |
