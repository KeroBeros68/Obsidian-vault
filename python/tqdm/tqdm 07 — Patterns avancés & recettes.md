#python #tqdm #patterns #avancé #recettes

## Barre de progression pour un entraînement ML
```python
from tqdm import tqdm, trange
import random

def train_epoch(model, loader):
    total_loss = 0.0
    with tqdm(loader, desc="Train", leave=False, unit="batch") as pbar:
        for batch_idx, (X, y) in enumerate(pbar):
            loss = model.step(X, y)
            total_loss += loss
            pbar.set_postfix({
                "loss":     f"{loss:.4f}",
                "avg_loss": f"{total_loss/(batch_idx+1):.4f}",
                "lr":       f"{model.lr:.2e}",
            })
    return total_loss / len(loader)

def train(model, train_loader, val_loader, epochs=10):
    epoch_bar = trange(epochs, desc="Epochs")
    best_val  = float("inf")

    for epoch in epoch_bar:
        train_loss = train_epoch(model, train_loader)
        val_loss   = validate(model, val_loader)
        if val_loss < best_val:
            best_val = val_loss
        epoch_bar.set_postfix({
            "train": f"{train_loss:.4f}",
            "val":   f"{val_loss:.4f}",
            "best":  f"{best_val:.4f}",
        })
```

## Barre de progression pour le traitement de fichiers
```python
from pathlib import Path
from tqdm import tqdm

def process_directory(root: str) -> None:
    files = list(Path(root).rglob("*.json"))   # lister d'abord pour connaître le total

    with tqdm(files, desc="Fichiers JSON", unit="fichier") as pbar:
        for path in pbar:
            pbar.set_description(f"Traite {path.name}")
            process_file(path)
```

## Wrapper générique réutilisable
```python
from tqdm import tqdm
from typing import Iterable, TypeVar

T = TypeVar("T")

def with_progress(
    iterable: Iterable[T],
    desc:     str       = "",
    total:    int | None = None,
    **kwargs,
) -> Iterable[T]:
    return tqdm(iterable, desc=desc, total=total, **kwargs)

for item in with_progress(my_data, desc="Traitement", total=len(my_data)):
    process(item)
```

## Désactiver conditionnellement selon l'environnement
```python
import os
from tqdm import tqdm

IS_CI = os.getenv("CI", "false").lower() == "true"

def progress(iterable, **kwargs):
    return tqdm(iterable, disable=IS_CI, **kwargs)

for item in progress(data, desc="Traitement"):
    process(item)
```

## tqdm comme décorateur
```python
from tqdm import tqdm
from functools import wraps
from typing import Callable

def tqdm_wrap(desc: str = "", **tqdm_kwargs):
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(items, *args, **kwargs):
            return [
                func(item, *args, **kwargs)
                for item in tqdm(items, desc=desc or func.__name__, **tqdm_kwargs)
            ]
        return wrapper
    return decorator

@tqdm_wrap(desc="Traitement", unit="item")
def process_all(items: list) -> list:
    return [item * 2 for item in items]
```

## Barre persistante pour des tâches longues
```python
from tqdm import tqdm

class LongTaskProgress:
    def __init__(self, total_steps: int, desc: str = ""):
        self._pbar = tqdm(total=total_steps, desc=desc, dynamic_ncols=True)

    def advance(self, n: int = 1, status: str = "") -> None:
        self._pbar.update(n)
        if status:
            self._pbar.set_postfix_str(status)

    def close(self) -> None:
        self._pbar.close()

    def __enter__(self): return self
    def __exit__(self, *args): self.close()

with LongTaskProgress(10, "Pipeline") as progress:
    progress.advance(1, "Chargement")
    load_data()
    progress.advance(3, "Preprocessing")
    preprocess()
    progress.advance(5, "Calcul")
    compute()
    progress.advance(1, "Export")
    export()
```

## Affichage stylisé avec rich
```python
from tqdm.rich import tqdm
import time

for i in tqdm(range(100), desc="[green]Traitement[/]"):
    time.sleep(0.02)
```

## Pipeline ETL complet
```python
from tqdm import tqdm
import json
from pathlib import Path

def etl_pipeline(input_dir: str, output_dir: str) -> dict:
    stats = {"processed": 0, "errors": 0}
    files = list(Path(input_dir).glob("*.json"))

    with tqdm(total=len(files), desc="ETL Pipeline", unit="fichier") as pbar:
        for path in files:
            pbar.set_description(f"ETL — {path.stem[:20]}")
            try:
                data     = json.loads(path.read_text())
                result   = transform(data)
                out_path = Path(output_dir) / path.name
                out_path.write_text(json.dumps(result))
                stats["processed"] += 1
            except Exception as e:
                tqdm.write(f"Erreur sur {path.name} : {e}")
                stats["errors"] += 1
            finally:
                pbar.update(1)
                pbar.set_postfix(stats)

    return stats
```
