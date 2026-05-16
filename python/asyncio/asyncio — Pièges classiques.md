#python #asyncio #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier `await` — coroutine jamais exécutée
```python
# ❌ Crée un objet coroutine et l'ignore immédiatement
async def main():
    asyncio.sleep(1)          # silencieux — rien ne se passe
    fetch("https://api.com")  # idem — warning "never awaited"

# ✅
async def main():
    await asyncio.sleep(1)
    result = await fetch("https://api.com")

# Détecter : python -W error::RuntimeWarning script.py
# Ou : PYTHONASYNCIODEBUG=1 python script.py
```

## 🪤 Piège 2 — await séquentiel au lieu de gather
```python
# ❌ Séquentiel — 3 secondes au lieu de 1
async def main():
    r1 = await fetch("url1")   # attend 1s
    r2 = await fetch("url2")   # attend 1s de plus
    r3 = await fetch("url3")   # attend 1s de plus

# ✅ Concurrent — 1 seconde
async def main():
    r1, r2, r3 = await asyncio.gather(
        fetch("url1"),
        fetch("url2"),
        fetch("url3"),
    )
```

## 🪤 Piège 3 — Code bloquant dans une coroutine — fige l'event loop
```python
import time, requests

# ❌ time.sleep et requests.get bloquent le thread entier — toutes les coroutines sont gelées
async def bad_fetch(url: str) -> str:
    time.sleep(2)                # ← BLOQUE tout
    return requests.get(url).text  # ← BLOQUE tout

# ✅ Utiliser asyncio.to_thread pour les fonctions synchrones bloquantes
async def good_fetch(url: str) -> str:
    return await asyncio.to_thread(requests.get, url)

# ✅ Ou utiliser une lib async native
import aiohttp
async def best_fetch(url: str) -> str:
    async with aiohttp.ClientSession() as s:
        async with s.get(url) as r:
            return await r.text()
```

## 🪤 Piège 4 — CancelledError avalée — task zombie
```python
# ❌ Avaler CancelledError → la task n'est jamais vraiment annulée
async def worker():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("Annulé")
        # OUBLI de raise → task continue à vivre (marquée done mais mal)

# ✅ Toujours re-raise
async def worker():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("Cleanup...")
        await cleanup()    # cleanup async autorisé
        raise              # ← OBLIGATOIRE
```

## 🪤 Piège 5 — Task exception silencieuse
```python
# ❌ Exception dans une task non awaitée → log silencieux seulement
async def main():
    task = asyncio.create_task(faulty_coro())
    await asyncio.sleep(5)    # task a crashé mais personne ne le sait

# ✅ Toujours awaiter les tasks
async def main():
    task = asyncio.create_task(faulty_coro())
    try:
        await task
    except Exception as e:
        print(f"Erreur : {e}")

# ✅ Ou ajouter un done_callback
task.add_done_callback(
    lambda t: t.exception() and print(f"Erreur : {t.exception()}")
)
```

## 🪤 Piège 6 — asyncio.run() dans une loop existante
```python
# ❌ RuntimeError : This event loop is already running
# (FastAPI, Jupyter, pytest-asyncio ont déjà une loop)
asyncio.run(main())

# ✅ Dans Jupyter
import nest_asyncio; nest_asyncio.patch()
await main()   # ou asyncio.run(main())

# ✅ Dans les tests
@pytest.mark.asyncio
async def test_something():
    result = await my_coroutine()
    assert result == expected

# ✅ Dans FastAPI — utiliser directement async def pour les routes
@app.get("/")
async def route():
    return await my_coro()
```

## 🪤 Piège 7 — Partager des objets non thread-safe avec to_thread
```python
# ❌ asyncio.to_thread tourne dans un vrai thread
# Les objets asyncio (Lock, Queue...) ne sont PAS thread-safe
async def bad():
    lock = asyncio.Lock()
    await asyncio.to_thread(some_sync_fn, lock)   # ❌ lock dans un thread

# ✅ Utiliser threading.Lock pour la communication thread/coroutine
# ✅ Ou asyncio.get_event_loop().call_soon_threadsafe() pour notifier la loop
```

## 🪤 Piège 8 — gather sans return_exceptions laisse des tasks pending
```python
# ❌ Si une task échoue, gather annule les autres mais les exceptions sont perdues
async def main():
    try:
        await asyncio.gather(task_a(), task_b(), task_c())
    except Exception as e:
        print(e)   # seulement la première exception — les autres sont perdues

# ✅ Capturer toutes les exceptions
results = await asyncio.gather(task_a(), task_b(), task_c(),
                                return_exceptions=True)
for r in results:
    if isinstance(r, Exception):
        print(f"Échec : {r}")
```

## 🪤 Piège 9 — queue.task_done() oublié → queue.join() bloque indéfiniment
```python
# ❌ task_done() oublié → join() attend pour toujours
async def consumer(q):
    item = await q.get()
    process(item)
    # OUBLI : q.task_done()

# ✅ task_done() dans finally
async def consumer(q):
    item = await q.get()
    try:
        await process(item)
    except Exception:
        pass
    finally:
        q.task_done()   # ← toujours, même en cas d'exception
```

## 🪤 Piège 10 — Créer une connexion par coroutine au lieu d'un pool
```python
# ❌ 100 requêtes → 100 connexions BDD → surcharge du serveur
async def get_user(user_id: int):
    conn = await asyncpg.connect("postgresql://...")  # nouvelle connexion !
    return await conn.fetchrow("SELECT * FROM users WHERE id=$1", user_id)

# ✅ Connection pool partagé
pool = await asyncpg.create_pool("postgresql://...", min_size=5, max_size=20)

async def get_user(user_id: int):
    async with pool.acquire() as conn:
        return await conn.fetchrow("SELECT * FROM users WHERE id=$1", user_id)
```

## 🪤 Piège 11 — asyncio.Lock non réentrant
```python
# ❌ asyncio.Lock n'est PAS réentrant — deadlock si même coroutine acquire deux fois
lock = asyncio.Lock()

async def outer():
    async with lock:
        await inner()   # ← deadlock si inner() essaie aussi d'acquérir lock

# ✅ Refactorer pour éviter la double acquisition, ou restructurer le code
```

## 🪤 Piège 12 — Utiliser asyncio pour du CPU-bound
```python
# ❌ asyncio ne parallélise PAS le CPU — un seul thread Python (GIL)
async def cpu_heavy():
    return sum(i*i for i in range(10_000_000))   # fige l'event loop 2-3s !

# ✅ Déléguer au ProcessPoolExecutor
async def cpu_heavy():
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as pool:
        return await loop.run_in_executor(pool,
            lambda: sum(i*i for i in range(10_000_000))
        )
```

## Récapitulatif rapide
| Piège | Solution |
|---|---|
| Oublier `await` | `python -W error::RuntimeWarning` pour détecter |
| await séquentiel | `asyncio.gather()` pour la concurrence |
| Code bloquant | `asyncio.to_thread()` ou lib async native |
| CancelledError avalée | Toujours `raise` après le cleanup |
| Task exception silencieuse | Awaiter ou `add_done_callback` |
| `asyncio.run()` dans une loop | `nest_asyncio` ou `pytest.mark.asyncio` |
| Objet asyncio dans un thread | `threading.Lock` ou `call_soon_threadsafe` |
| `gather` sans `return_exceptions` | Ajouter `return_exceptions=True` |
| `task_done()` oublié | Mettre dans `finally` |
| Connexion BDD par coroutine | Connection pool (`asyncpg.create_pool`) |
| Lock non réentrant | Éviter la double acquisition |
| CPU-bound dans asyncio | `ProcessPoolExecutor` via `run_in_executor` |
