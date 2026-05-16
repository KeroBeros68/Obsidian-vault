#python #asyncio #event-loop #coroutines #modèle #concurrence

## Le problème que asyncio résout
```
Deux façons d'attendre :

  Bloquant (synchrone) :
    thread 1 : ──[req A]──────────[req B]──────────[req C]──
                           ↑ attend      ↑ attend
                         3 req × 1s = 3s total

  Non-bloquant (asyncio) :
    event loop : ──[req A]──[req B]──[req C]──[A ok]──[B ok]──[C ok]──
                             ↑ lance tout en parallèle
                           ≈ 1s total (durée de la req la plus lente)
```

asyncio est fait pour les **tâches I/O-bound** (réseau, fichiers, BDD) — pas pour les tâches CPU-bound (calcul pur → utiliser `multiprocessing` ou `concurrent.futures`).

## Le modèle — un seul thread, coopératif
```
Thread unique
    │
    ▼
┌─────────────────────────────┐
│         Event Loop          │
│                             │
│  File de tâches prêtes      │
│  ┌────┐ ┌────┐ ┌────┐      │
│  │ T1 │ │ T2 │ │ T3 │ ...  │
│  └────┘ └────┘ └────┘      │
└─────────────────────────────┘
         │
         ▼
    Sélecteur I/O (epoll / kqueue / IOCP)
    surveille les fds en attente

Chaque tâche tourne jusqu'au prochain `await`
→ elle rend la main à l'event loop
→ l'event loop passe à la prochaine tâche prête
```

## Coroutine — unité de base
```python
import asyncio

# Une coroutine est définie avec async def
async def fetch(url: str) -> str:
    print(f"Début : {url}")
    await asyncio.sleep(1)      # ← point de suspension — rend la main
    print(f"Fin : {url}")
    return f"résultat de {url}"

# Appeler la coroutine NE l'exécute PAS — crée un objet coroutine
coro = fetch("https://example.com")
print(type(coro))   # <class 'coroutine'>

# Pour l'exécuter, il faut l'awaiter ou la passer à asyncio.run()
result = asyncio.run(fetch("https://example.com"))
```

## Coroutine vs fonction normale vs thread
| | Fonction sync | Thread | Coroutine (asyncio) |
|---|---|---|---|
| Exécution | Bloquante | Parallèle (OS) | Coopérative (event loop) |
| Mémoire | faible | ~1 Mo/thread | ~2 Ko/coroutine |
| Commutation | — | OS (préemptive) | `await` (volontaire) |
| Sécurité | — | verrous requis | pas de race conditions* |
| Idéal pour | calcul | CPU-bound | I/O-bound |

`*` dans un seul thread — les accès partagés restent sûrs entre `await`

## Trois niveaux d'abstraction asyncio
```
Niveau 3 — Haut niveau (recommandé)
  asyncio.run(), await, asyncio.gather(), asyncio.create_task()

Niveau 2 — Intermédiaire
  asyncio.get_event_loop(), loop.run_until_complete()
  Task, Future, Shield

Niveau 1 — Bas niveau (rarement nécessaire)
  Transport, Protocol, loop.call_soon(), loop.call_later()
```

> [!tip] asyncio ≠ parallélisme
> asyncio est de la **concurrence** (une tâche à la fois, mais on alterne rapidement), pas du parallélisme (plusieurs tâches simultanées sur plusieurs cœurs).
> Pour du CPU-bound : `concurrent.futures.ProcessPoolExecutor` ou `multiprocessing`.

> [!important] Prérequis
> asyncio repose sur le concept de **générateur coroutine** — si `yield` et `send()` sont flous, lire d'abord [[Générateurs 06 — send, throw & close]].
