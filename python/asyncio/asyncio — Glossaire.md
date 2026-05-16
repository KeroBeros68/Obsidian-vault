#python #asyncio #glossaire #référence

| Terme | Définition |
|---|---|
| **event loop** | Boucle centrale qui ordonnance et exécute les coroutines et callbacks |
| **coroutine** | Fonction définie avec `async def` — suspendue à chaque `await` |
| **Awaitable** | Objet qu'on peut `await` — coroutine, Task, ou Future |
| **await** | Suspend la coroutine courante et rend la main à l'event loop |
| **Task** | Wrapper autour d'une coroutine planifiée sur l'event loop |
| **Future** | Résultat futur positionné manuellement — primitive bas niveau |
| **asyncio.run()** | Crée une event loop, exécute une coroutine, ferme proprement |
| **asyncio.create_task()** | Planifie une coroutine comme Task concurrente immédiatement |
| **asyncio.gather()** | Lance N awaitables en parallèle, retourne les résultats dans l'ordre d'entrée |
| **asyncio.wait()** | Attend N tasks avec contrôle fin (FIRST_COMPLETED, timeout…) |
| **asyncio.as_completed()** | Itère sur les tasks dans l'ordre de completion |
| **asyncio.sleep()** | Suspend la coroutine pendant N secondes — rend la main à la loop |
| **asyncio.Queue** | File thread-safe pour la communication entre coroutines |
| **asyncio.Lock** | Mutex — exclusion mutuelle entre coroutines |
| **asyncio.Event** | Signal one-shot — `set()` réveille tous les `wait()` |
| **asyncio.Semaphore** | Limite le nombre de coroutines accédant simultanément à une ressource |
| **asyncio.TaskGroup** | Gestionnaire de contexte pour un groupe de tasks (Python 3.11+) |
| **asyncio.timeout()** | Context manager de timeout (Python 3.11+) |
| **asyncio.wait_for()** | Timeout sur une coroutine avec annulation automatique (3.7+) |
| **asyncio.shield()** | Protège une coroutine de l'annulation |
| **asyncio.to_thread()** | Exécute une fonction bloquante dans un thread (Python 3.9+) |
| **run_in_executor()** | Exécute une fonction dans un thread ou process pool |
| **CancelledError** | Exception levée dans une coroutine annulée — toujours re-raise |
| **ExceptionGroup** | Groupe d'exceptions (Python 3.11+) — levé par TaskGroup |
| **async for** | Itération sur un async iterable |
| **async with** | Context manager asynchrone — `__aenter__` / `__aexit__` |
| **@asynccontextmanager** | Décorateur pour créer un async context manager depuis un générateur |
| **ASGI** | Async Server Gateway Interface — protocole de FastAPI, Starlette |
| **I/O-bound** | Tâche limitée par les entrées/sorties — idéale pour asyncio |
| **CPU-bound** | Tâche limitée par le calcul — nécessite multiprocessing ou threads |
| **backpressure** | Mécanisme limitant la production quand la consommation est trop lente |
| **fan-out / fan-in** | Distribuer N tâches (fan-out) puis rassembler les résultats (fan-in) |
| **circuit breaker** | Pattern qui coupe les appels en cas de défaillances répétées |
| **debounce** | N'exécuter une fonction qu'après un délai sans nouvel appel |
| **throttle** | Limiter la fréquence d'exécution d'une fonction |
| **sentinel** | Objet unique signalant la fin d'une queue (`None` ou objet spécial) |
