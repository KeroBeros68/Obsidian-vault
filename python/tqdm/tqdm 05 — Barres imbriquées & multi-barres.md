#python #tqdm #imbriquées #multi #position #nested

## Barres imbriquées — epochs et batches
```python
from tqdm import tqdm
import time

epochs  = 5
batches = 20

for epoch in tqdm(range(epochs), desc="Epochs", position=0, leave=True):
    for batch in tqdm(range(batches), desc=f"  Epoch {epoch+1}", position=1, leave=False):
        time.sleep(0.02)

# Affichage :
# Epochs:  60%|██████████          | 3/5 [00:02<00:01]
#   Epoch 4:  45%|████████▊        | 9/20 [00:00<00:00]
```

## Position explicite — plusieurs barres simultanées
```python
from tqdm import tqdm
import threading
import time

def worker(worker_id: int, n: int) -> None:
    for i in tqdm(range(n),
                  desc=f"Worker {worker_id}",
                  position=worker_id,
                  leave=True):
        time.sleep(0.05)

threads = [threading.Thread(target=worker, args=(i, 50)) for i in range(4)]
for t in threads: t.start()
for t in threads: t.join()
```

## tqdm.contrib.concurrent — workers parallèles
```python
from tqdm.contrib.concurrent import thread_map, process_map
import time

def process_item(x: int) -> int:
    time.sleep(0.1)
    return x * x

# thread_map — threading, I/O-bound
results = thread_map(process_item, range(20), max_workers=4, desc="Threads")

# process_map — multiprocessing, CPU-bound
results = process_map(process_item, range(20), max_workers=4, desc="Processus",
                      chunksize=2)
```

## tqdm.contrib.itertools
```python
from tqdm.contrib import tenumerate, tzip, tmap

for i, item in tenumerate(my_list, desc="Enum"):
    pass

for a, b in tzip(list1, list2, desc="Zip"):
    pass

results = list(tmap(fn, items, desc="Map"))
```

## Barres dans des threads — thread-safe
```python
from tqdm import tqdm
import threading

lock = threading.Lock()

def safe_update(pbar, n=1):
    with lock:
        pbar.update(n)

# Ou utiliser tqdm.contrib.concurrent qui gère ça automatiquement
```

> [!warning] `position` et `leave` sont liés
> Toujours mettre `leave=False` sur les barres internes, `leave=True` sur la barre externe — sinon les barres résiduelles s'accumulent dans le terminal.
