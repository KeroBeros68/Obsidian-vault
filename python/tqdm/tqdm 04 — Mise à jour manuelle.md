#python #tqdm #manuel #update #postfix #context-manager

## Context manager — contrôle total
```python
from tqdm import tqdm
import time

with tqdm(total=100, desc="Traitement") as pbar:
    for i in range(10):
        time.sleep(0.1)
        pbar.update(10)   # avancer de 10 unités
# 100%|██████████████████████| 100/100 [00:01<00:00, 99.87it/s]
```

## update() — avancer la barre
```python
from tqdm import tqdm

with tqdm(total=1000, unit="B", unit_scale=True) as pbar:
    downloaded = 0
    for chunk in stream_data():
        downloaded += len(chunk)
        pbar.update(len(chunk))
```

## set_description() — changer le label en cours de route
```python
from tqdm import tqdm

stages = ["Chargement", "Nettoyage", "Calcul", "Export"]

with tqdm(total=len(stages)) as pbar:
    for stage in stages:
        pbar.set_description(f"Étape : {stage}")
        do_stage(stage)
        pbar.update(1)
```

## set_postfix() — afficher des métriques en temps réel
```python
from tqdm import tqdm
import random

with tqdm(range(100), desc="Entraînement") as pbar:
    for epoch in pbar:
        loss     = random.random()
        accuracy = 0.5 + random.random() * 0.5

        pbar.set_postfix({
            "loss":  f"{loss:.4f}",
            "acc":   f"{accuracy:.2%}",
            "epoch": epoch,
        })
        train_one_epoch()

# Entraînement:  42%|████████▌         | 42/100 [00:04<00:05, loss=0.2341, acc=87.32%, epoch=42]
```

## set_postfix_str() — postfix texte libre
```python
from tqdm import tqdm

with tqdm(range(50)) as pbar:
    for i in pbar:
        pbar.set_postfix_str(f"item courant = {i*2}")
```

## reset() et clear()
```python
from tqdm import tqdm

pbar = tqdm(total=100)

pbar.update(50)
pbar.reset()       # remet à zéro — réutiliser la même barre
pbar.update(30)

pbar.clear()       # efface temporairement l'affichage (pour un print propre)
print("Message propre sans barre")
pbar.display()     # réaffiche la barre
pbar.close()       # fermer proprement
```

## write() — afficher du texte sans casser la barre
```python
from tqdm import tqdm

with tqdm(range(100)) as pbar:
    for i in pbar:
        if i % 10 == 0:
            tqdm.write(f"Checkpoint à {i}")   # ✅ s'insère proprement au-dessus
            # print(f"...")                    # ❌ casse l'affichage de la barre
```

## Reprendre depuis un point de départ
```python
from tqdm import tqdm

with tqdm(total=1000, initial=500, desc="Reprise") as pbar:
    for i in range(500):
        pbar.update(1)
# Commence à 50% directement
```

> [!warning] Toujours fermer la barre
> Utiliser le context manager (`with tqdm(...) as pbar`) garantit l'appel à `pbar.close()` même en cas d'exception. Une barre non fermée peut laisser le curseur terminal dans un état incohérent.
