#python #tqdm #pandas #asyncio #jupyter #intégrations

## Pandas — progress_apply
```python
import pandas as pd
from tqdm import tqdm

# Activer l'intégration Pandas — à appeler UNE seule fois
tqdm.pandas(desc="Traitement")

df = pd.DataFrame({"val": range(10_000)})

df["double"] = df["val"].progress_apply(lambda x: x * 2)
df["str"]    = df["val"].progress_map(str)
df["result"] = df.progress_apply(lambda row: row["val"] * 2, axis=1)
```

## Pandas — groupby avec progression
```python
import pandas as pd
from tqdm import tqdm

tqdm.pandas()

df = pd.DataFrame({
    "group": ["A", "B", "A", "B", "A"] * 1000,
    "val":   range(5000),
})

result = df.groupby("group")["val"].progress_apply(lambda x: x.sum())
```

## asyncio — tqdm async
```python
import asyncio
from tqdm.asyncio import tqdm as atqdm

async def fetch(i: int) -> int:
    await asyncio.sleep(0.1)
    return i * 2

async def main() -> None:
    # tqdm sur as_completed
    results = []
    async for result in atqdm(
        asyncio.as_completed([fetch(i) for i in range(20)]),
        total=20,
        desc="Requêtes async",
    ):
        results.append(await result)

    # gather avec barre
    results = await atqdm.gather(
        *[fetch(i) for i in range(20)],
        desc="Gather async",
    )

asyncio.run(main())
```

## Jupyter Notebook — widget interactif
```python
from tqdm.notebook import tqdm   # widget HTML interactif
# ou
from tqdm.auto import tqdm       # auto-sélectionne notebook si dans Jupyter

import time

for i in tqdm(range(100), desc="Notebook"):
    time.sleep(0.05)
# Affiche un widget interactif coloré
```

## Intégration avec logging
```python
import logging
from tqdm import tqdm
from tqdm.contrib.logging import logging_handler

logger = logging.getLogger(__name__)

with logging_handler(logger):
    for i in tqdm(range(100)):
        if i % 20 == 0:
            logger.info(f"Checkpoint {i}")   # ✅ propre, ne casse pas la barre
```

## tqdm avec requests — téléchargement
```python
import requests
from tqdm import tqdm

def download(url: str, dest: str) -> None:
    response = requests.get(url, stream=True)
    total    = int(response.headers.get("content-length", 0))

    with open(dest, "wb") as f, tqdm(
        total=total, unit="B", unit_scale=True,
        unit_divisor=1024, desc=dest,
    ) as pbar:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
            pbar.update(len(chunk))
```

## tqdm avec aiohttp — téléchargement async
```python
import asyncio
import aiohttp
from tqdm.asyncio import tqdm as atqdm

async def async_download(url: str, dest: str) -> None:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            total = int(response.headers.get("content-length", 0))

            with open(dest, "wb") as f:
                with atqdm(total=total, unit="B", unit_scale=True,
                           unit_divisor=1024, desc=dest) as pbar:
                    async for chunk in response.content.iter_chunked(8192):
                        f.write(chunk)
                        pbar.update(len(chunk))
```

## tqdm avec multiprocessing
```python
from tqdm.contrib.concurrent import process_map
from multiprocessing import Pool
from tqdm import tqdm

def heavy_compute(x: int) -> int:
    return x ** 2

# process_map — barre automatique
results = process_map(heavy_compute, range(100), max_workers=4, chunksize=5)

# Avec Pool manuel
with Pool(4) as pool:
    results = list(tqdm(
        pool.imap(heavy_compute, range(100), chunksize=5),
        total=100, desc="Multiprocessing",
    ))
```

> [!tip] `tqdm.auto` dans tous les contextes
> `from tqdm.auto import tqdm` est le seul import dont tu as besoin pour la majorité des cas.
