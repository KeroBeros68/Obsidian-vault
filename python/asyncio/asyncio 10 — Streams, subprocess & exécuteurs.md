#python #asyncio #streams #subprocess #executor #threading

## asyncio Streams — TCP/IP asynchrone
```python
import asyncio

# CLIENT TCP async
async def tcp_client(host: str, port: int) -> None:
    reader, writer = await asyncio.open_connection(host, port)
    try:
        writer.write(b"Hello\n")
        await writer.drain()              # flush le buffer

        data = await reader.read(1024)    # lire jusqu'à 1024 bytes
        line = await reader.readline()    # lire jusqu'au \n
        print(f"Reçu : {data.decode()}")
    finally:
        writer.close()
        await writer.wait_closed()

# SERVEUR TCP async
async def handle_client(reader: asyncio.StreamReader,
                         writer: asyncio.StreamWriter) -> None:
    addr = writer.get_extra_info("peername")
    print(f"Connexion de {addr}")
    data = await reader.read(100)
    writer.write(data)            # echo
    await writer.drain()
    writer.close()

async def main() -> None:
    server = await asyncio.start_server(handle_client, "127.0.0.1", 8888)
    async with server:
        await server.serve_forever()
```

## asyncio subprocess — lancer des processus
```python
import asyncio

async def run_command(cmd: str) -> tuple[str, str, int]:
    proc = await asyncio.create_subprocess_shell(
        cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )
    stdout, stderr = await proc.communicate()   # attend la fin + lit tout
    return stdout.decode(), stderr.decode(), proc.returncode

async def main() -> None:
    out, err, code = await run_command("ls -la")
    print(f"Sortie : {out}")
    print(f"Code retour : {code}")

    # Ou avec exec (plus sûr que shell pour les arguments)
    proc = await asyncio.create_subprocess_exec(
        "python3", "-c", "print('hello')",
        stdout=asyncio.subprocess.PIPE,
    )
    stdout, _ = await proc.communicate()
```

## run_in_executor — code bloquant dans un thread/process
```python
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def blocking_io(path: str) -> str:
    """Fonction synchrone bloquante — ex: librairie sans support async."""
    time.sleep(2)
    return open(path).read()

def cpu_intensive(n: int) -> int:
    """Calcul pur — bénéficie d'un ProcessPool."""
    return sum(i*i for i in range(n))

async def main() -> None:
    loop = asyncio.get_running_loop()

    # Thread pool (défaut) — pour I/O bloquant
    result = await loop.run_in_executor(None, blocking_io, "/etc/hosts")

    # Thread pool explicite
    with ThreadPoolExecutor(max_workers=4) as pool:
        result = await loop.run_in_executor(pool, blocking_io, "/etc/hosts")

    # Process pool — pour CPU-bound (contourne le GIL)
    with ProcessPoolExecutor(max_workers=4) as pool:
        result = await loop.run_in_executor(pool, cpu_intensive, 10_000_000)

    # asyncio.to_thread — raccourci Python 3.9+
    result = await asyncio.to_thread(blocking_io, "/etc/hosts")
```

## asyncio.to_thread — l'API moderne (Python 3.9+)
```python
import asyncio
import requests   # lib synchrone

async def fetch_sync_lib(url: str) -> str:
    # Exécuter requests.get dans un thread → ne bloque pas l'event loop
    response = await asyncio.to_thread(requests.get, url)
    return response.text

async def main() -> None:
    results = await asyncio.gather(
        fetch_sync_lib("https://example.com/1"),
        fetch_sync_lib("https://example.com/2"),
        fetch_sync_lib("https://example.com/3"),
    )
```

## Quand utiliser quoi ?
| Cas | Solution |
|---|---|
| I/O async natif (aiohttp, asyncpg) | `await` directement |
| Lib sync I/O (requests, psycopg2) | `asyncio.to_thread()` |
| Calcul CPU intense | `run_in_executor(ProcessPoolExecutor)` |
| Processus externe | `asyncio.create_subprocess_exec()` |
| Réseau TCP bas niveau | `asyncio.open_connection()` |

> [!warning] run_in_executor ne magie pas le GIL
> `ThreadPoolExecutor` contourne l'event loop (threads libèrent le GIL en I/O), mais pas le GIL pour le calcul Python pur.
> Pour du vrai parallélisme CPU : `ProcessPoolExecutor`.
