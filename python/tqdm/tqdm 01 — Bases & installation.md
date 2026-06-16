#python #tqdm #installation #bases

## Installation
```bash
pip install tqdm

# Avec support notebook Jupyter
pip install tqdm ipywidgets
jupyter nbextension enable --py widgetsnbextension

# Avec support asyncio
pip install tqdm[asyncio]
```

## Import
```python
from tqdm import tqdm                    # CLI standard
from tqdm.auto import tqdm               # ✅ auto-détecte CLI vs notebook
from tqdm.notebook import tqdm           # notebook uniquement (widget)
from tqdm.asyncio import tqdm as atqdm   # coroutines async
```

> [!tip] Toujours utiliser `tqdm.auto`
> `from tqdm.auto import tqdm` sélectionne automatiquement le bon backend selon l'environnement (terminal, Jupyter, notebook). C'est le choix par défaut pour tout code qui pourrait tourner dans les deux contextes.

## Concept — envelopper un itérable
```python
from tqdm import tqdm
import time

# Sans tqdm
for i in range(100):
    time.sleep(0.05)

# Avec tqdm — même code, juste enveloppé
for i in tqdm(range(100)):
    time.sleep(0.05)
# 100%|████████████████████| 100/100 [00:05<00:00, 19.97it/s]
```

## Anatomie de la barre
```
 42%|████████████▌             | 420/1000 [00:21<00:29, 19.97it/s]
 │   │                          │    │      │      │     └── vitesse (it/s)
 │   │                          │    │      │      └──────── temps restant estimé
 │   │                          │    │      └─────────────── temps écoulé
 │   │                          │    └────────────────────── total
 │   │                          └─────────────────────────── progression
 │   └────────────────────────────────────────────────────── barre visuelle
 └────────────────────────────────────────────────────────── pourcentage
```

## Vérifier la version
```python
import tqdm
print(tqdm.__version__)   # ex: "4.66.4"
```
