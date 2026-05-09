#pydantic #settings #env #config #dotenv

## Installation

```bash
pip install pydantic-settings
```

## BaseSettings — lire les variables d'environnement

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
        extra="ignore",
    )

    app_name:      str       = "Mon App"
    debug:         bool      = False
    db_url:        str       = "sqlite:///./dev.db"
    secret_key:    str       # ← obligatoire, pas de défaut
    max_workers:   int       = 4
    allowed_hosts: list[str] = ["localhost"]

settings = Settings()
print(settings.db_url)
```

## Fichier .env correspondant

```dotenv
APP_NAME=Production App
DEBUG=false
DB_URL=postgresql://user:pass@localhost/prod
SECRET_KEY=super-secret-key-ici
MAX_WORKERS=8
ALLOWED_HOSTS=["api.example.com","www.example.com"]
```

## Priorité des sources (haute → basse)

```
1. Variables d'environnement système  (export VAR=val)
2. Variables du fichier .env
3. Valeurs par défaut dans la classe
```

## Préfixe de variables

```python
class DatabaseSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="DB_")
    host:     str = "localhost"
    port:     int = 5432
    password: str

# Lit DB_HOST, DB_PORT, DB_PASSWORD
```

## Plusieurs fichiers .env

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=(".env", ".env.local", ".env.production"),
        # Les fichiers de droite ont priorité sur ceux de gauche
    )
```

## Nested settings — double underscore

```python
class DBConfig(BaseSettings):
    host: str = "localhost"
    port: int = 5432

class AppSettings(BaseSettings):
    db:    DBConfig = DBConfig()
    debug: bool     = False

# Variable d'env : DB__HOST=prod.db (double underscore)
```

## Pattern recommandé — singleton FastAPI

```python
from functools import lru_cache

@lru_cache()
def get_settings() -> Settings:
    return Settings()   # lu une seule fois au démarrage

# Dans FastAPI
from fastapi import Depends

@app.get("/info")
def info(settings: Settings = Depends(get_settings)):
    return {"app": settings.app_name}
```

> [!tip] lru_cache sur get_settings Sans `@lru_cache`, `Settings()` relit le `.env` à chaque requête → lent. Avec le cache, c'est lu une seule fois au démarrage.

> [!warning] Ne jamais committer le .env Toujours ajouter `.env` au `.gitignore`. Versionner `.env.example` avec des valeurs fictives.
