#python #asyncio #asyncio-run #event-loop #entrée

## asyncio.run — le point d'entrée standard (Python 3.7+)
```python
import asyncio

async def main() -> None:
    print("Début")
    await asyncio.sleep(1)
    print("Fin")

# Point d'entrée — crée l'event loop, l'exécute, la ferme proprement
asyncio.run(main())

# ✅ asyncio.run() :
#   1. Crée une nouvelle event loop
#   2. Exécute la coroutine jusqu'à completion
#   3. Ferme l'event loop (annule les tâches pending, ferme les exécuteurs)
#   4. Appelle loop.close()
```

## asyncio.run — comportement complet
```python
import asyncio

async def main() -> None:
    # Tout le code async de l'application vit ici
    tasks = [
        asyncio.create_task(fetch("url1")),
        asyncio.create_task(fetch("url2")),
    ]
    results = await asyncio.gather(*tasks)
    print(results)

if __name__ == "__main__":
    asyncio.run(main())          # ← toujours dans le if __name__ == "__main__"
    # ou asyncio.run(main(), debug=True)  ← active les warnings et logs détaillés
```

## Mode debug — détecter les problèmes
```python
# Mode debug active :
#   - Warnings pour les coroutines jamais awaitées
#   - Logs pour les callbacks lents (>100ms)
#   - Vérification que les await sont bien dans des coroutines

asyncio.run(main(), debug=True)

# Ou via variable d'environnement
# PYTHONASYNCIODEBUG=1 python script.py
```

## Accéder à l'event loop courante
```python
import asyncio

async def current_loop_info() -> None:
    loop = asyncio.get_event_loop()        # loop courante (dépréciée hors async)
    loop = asyncio.get_running_loop()      # ✅ à préférer — lève RuntimeError si pas de loop

    # Planifier depuis du code sync (thread différent)
    asyncio.get_event_loop().call_soon_threadsafe(callback)
```

## Intégration dans du code synchrone existant
```python
import asyncio

# Cas 1 — on contrôle le point d'entrée → asyncio.run()
asyncio.run(main())

# Cas 2 — dans un contexte qui a déjà une loop (Jupyter, FastAPI, tests)
# ❌ asyncio.run() lève RuntimeError si une loop tourne déjà

# ✅ Dans Jupyter ou IPython — utiliser nest_asyncio
import nest_asyncio
nest_asyncio.patch()
asyncio.run(main())   # fonctionne maintenant

# ✅ Dans des tests — pytest-asyncio
# pip install pytest-asyncio
import pytest

@pytest.mark.asyncio
async def test_fetch() -> None:
    result = await fetch("https://example.com")
    assert result is not None
```

## asyncio.run dans les scripts vs bibliothèques
```python
# SCRIPT — asyncio.run() est approprié
if __name__ == "__main__":
    asyncio.run(main())

# BIBLIOTHÈQUE — ne jamais appeler asyncio.run()
# → l'appelant gère sa propre event loop
# → exposer des coroutines, pas des appels bloquants

# ❌ Dans une lib
def get_data() -> str:
    return asyncio.run(fetch_data())   # force l'appelant async à crasher

# ✅ Dans une lib
async def get_data() -> str:
    return await fetch_data()          # l'appelant await
```

> [!tip] Une seule loop par process
> `asyncio.run()` crée et ferme une loop complète. L'appeler deux fois crée deux loops séparées. Mettre tout le code async sous un seul `asyncio.run(main())`.

> [!warning] asyncio.run() ne peut pas être appelé si une loop tourne déjà
> Dans FastAPI, les tests pytest-asyncio, ou Jupyter, une loop tourne déjà.
> → Utiliser `await` directement, `pytest.mark.asyncio`, ou `nest_asyncio`.
