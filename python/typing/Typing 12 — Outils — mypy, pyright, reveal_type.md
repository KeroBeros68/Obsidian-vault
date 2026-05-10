#python #typing #mypy #pyright #outils #linter

## mypy — le vérificateur de référence

bash

```bash
pip install mypy

# Vérifier un fichier
mypy script.py

# Vérifier tout un projet
mypy src/

# Avec configuration stricte
mypy --strict src/
```

## pyproject.toml — configuration mypy

toml

```toml
[tool.mypy]
python_version = "3.11"
strict         = true        # active tous les checks stricts
warn_unused_ignores = true
ignore_missing_imports = true   # si certaines libs n'ont pas de stubs

# Par module
[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false   # moins strict pour les tests
```

## pyright / pylance — alternative Microsoft (rapide)

bash

```bash
pip install pyright

pyright src/
```

json

```json
// pyrightconfig.json
{
  "include": ["src"],
  "typeCheckingMode": "strict",   // "off" | "basic" | "standard" | "strict"
  "pythonVersion": "3.11"
}
```

## reveal_type — inspecter le type inféré


```python
# reveal_type est une fonction magique du type checker (pas à l'exécution)
x = [1, 2, 3]
reveal_type(x)            # mypy : Revealed type is "list[int]"

def add(a: int, b: int) -> int:
    return a + b

result = add(1, 2)
reveal_type(result)       # Revealed type is "int"
```

## # type: ignore — supprimer une erreur mypy


```python
x: int = "hello"                  # ❌ mypy error
x: int = "hello"  # type: ignore  # ✅ mypy ignore cette ligne

# Cibler un code d'erreur précis
x: int = "hello"  # type: ignore[assignment]
```

## cast — forcer un type sans vérification


```python
from typing import cast

# Quand tu sais mieux que mypy ce que c'est
raw = get_data()                       # → Any
name: str = cast(str, raw["name"])     # ✅ mypy accepte
# ⚠️ cast ne fait rien au runtime — c'est une indication au type checker uniquement
```

## Stubs — annotations pour les librairies sans types

bash

```bash
# Installer les stubs officiels (ex: requests, boto3...)
pip install types-requests
pip install boto3-stubs

# Mypy cherche automatiquement les stubs dans le projet
# Dossier : stubs/mylib.pyi  ou  mylib.pyi à côté du module
```

## Niveaux de rigueur mypy — du plus permissif au plus strict

```
pas d'option         → vérifie seulement les erreurs évidentes
--check-untyped-defs → vérifie aussi les fonctions non annotées
--disallow-untyped-defs → exige les annotations partout
--strict             → tout le monde doit être annoté + checks avancés
```

> [!tip] Commencer progressivement Ne pas activer `--strict` d'un coup sur un gros projet. Activer les options une par une en commençant par `--check-untyped-defs`.

> [!warning] mypy vs pyright — différences subtiles mypy et pyright ne s'accordent pas toujours sur les cas limites. En cas de conflit, privilégier mypy (plus conservateur) ou pyright (plus rapide et intégré à VSCode/Pylance).
