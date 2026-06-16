#python #tqdm #paramètres #personnalisation #style

## Paramètres principaux
```python
from tqdm import tqdm

pbar = tqdm(
    iterable,
    desc        = "Description",    # label à gauche de la barre
    total       = 1000,             # nombre total d'itérations
    unit        = "it",             # unité affichée (it/s par défaut)
    unit_scale  = True,             # 1000 → 1k, 1_000_000 → 1M
    unit_divisor= 1024,             # diviseur pour unit_scale (1000 par défaut, 1024 pour octets)
    ncols       = 80,               # largeur fixe de la barre (None = auto)
    miniters    = 1,                # mise à jour minimum tous les N items
    mininterval = 0.1,              # mise à jour minimum toutes les N secondes
    maxinterval = 10.0,             # mise à jour maximum toutes les N secondes
    ascii       = False,            # True = caractères ASCII purs (compatibilité)
    colour      = "green",          # couleur de la barre (green, red, blue, yellow...)
    dynamic_ncols = True,           # adapte la largeur au terminal en temps réel
    leave       = True,             # True = barre reste après completion, False = efface
    position    = 0,                # position verticale (pour barres imbriquées)
    file        = sys.stderr,       # sortie (stderr par défaut)
    disable     = False,            # True = désactive toute la barre (mode silencieux)
    smoothing   = 0.3,              # lissage de la vitesse (0=moyenne totale, 1=instantané)
    bar_format  = None,             # format custom (voir ci-dessous)
    initial     = 0,                # valeur de départ (utile pour reprendre)
    postfix     = None,             # dict de stats affichées à droite
)
```

## bar_format — format entièrement custom
```python
from tqdm import tqdm

# Variables disponibles dans bar_format :
# {l_bar} {bar} {r_bar} {n} {n_fmt} {total} {total_fmt}
# {percentage} {elapsed} {elapsed_s} {remaining} {remaining_s}
# {rate} {rate_fmt} {rate_noinv} {rate_noinv_fmt}
# {postfix} {unit} {desc}

for i in tqdm(range(100),
    bar_format="{desc}: {percentage:.1f}%|{bar}| {n}/{total} [{elapsed}<{remaining}]"):
    pass

# Barre minimaliste — juste le compteur
for i in tqdm(range(100), bar_format="{n}/{total}"):
    pass
```

## Couleurs
```python
from tqdm import tqdm

# Couleurs supportées : green, red, blue, yellow, cyan, magenta, white
for i in tqdm(range(100), colour="green",  desc="OK"):      pass
for i in tqdm(range(100), colour="red",    desc="Erreur"):  pass
for i in tqdm(range(100), colour="yellow", desc="Attente"): pass
for i in tqdm(range(100), colour="#FF6B6B", desc="Custom"): pass   # hex
```

## unit_scale — affichage d'octets
```python
from tqdm import tqdm

total_bytes = 100 * 1024 * 1024   # 100 Mo

with tqdm(
    total       = total_bytes,
    unit        = "B",
    unit_scale  = True,
    unit_divisor= 1024,
    desc        = "Téléchargement",
) as pbar:
    for chunk in download_chunks():
        pbar.update(len(chunk))
# Téléchargement: 100%|██████| 100M/100M [00:10<00:00, 9.87MB/s]
```

## disable — mode silencieux conditionnel
```python
import os
from tqdm import tqdm

verbose = os.getenv("VERBOSE", "1") == "1"

for i in tqdm(range(100), disable=not verbose):
    pass

# Dans des tests — toujours désactiver
for i in tqdm(range(100), disable=True):
    pass
```

## smoothing — contrôler le lissage de la vitesse
```python
from tqdm import tqdm

# smoothing=0   → vitesse moyenne totale (stable, lente à réagir)
# smoothing=1   → vitesse instantanée (réactive, fluctuante)
# smoothing=0.3 → équilibre (défaut)

for i in tqdm(range(100), smoothing=0):    pass   # vitesse stable
for i in tqdm(range(100), smoothing=0.9):  pass   # très réactif
```

> [!tip] `leave=False` pour les sous-barres
> Dans une boucle imbriquée, mettre `leave=False` sur la barre interne pour qu'elle disparaisse après chaque epoch. La barre externe reste.
