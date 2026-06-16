#python #fire #cli #arguments #flags #types

## Arguments positionnels et nommés

```python
import fire

def greet(name: str, greeting: str = "Hello"):
    return f"{greeting}, {name}!"

fire.Fire(greet)
```

```bash
# Positionnels — dans l'ordre des paramètres
python script.py Kevin            # → Hello, Kevin!
python script.py Kevin Bonjour    # → Bonjour, Kevin!

# Nommés — ordre libre
python script.py --name=Kevin --greeting=Bonjour
python script.py --greeting Bonjour --name Kevin   # équivalent
```

## Coercition de types — via annotations

Fire coerce automatiquement les chaînes CLI vers le type annoté.

```python
def process(n: int, ratio: float, verbose: bool = False):
    print(n, ratio, verbose)

fire.Fire(process)
```

```bash
python script.py --n=10 --ratio=0.5           # int, float ✅
python script.py --n=10 --ratio=0.5 --verbose  # verbose=True ✅
```

| Annotation | Entrée CLI | Résultat |
|---|---|---|
| `int` | `"42"` | `42` |
| `float` | `"3.14"` | `3.14` |
| `str` | `"hello"` | `"hello"` |
| `bool` | `--flag` | `True` |
| `bool` | `--noflag` | `False` |
| `list[int]` | `"[1,2,3]"` | `[1, 2, 3]` |

> [!info] Sans annotation
> Fire utilise une heuristique : il tente d'évaluer la chaîne comme un littéral Python (`ast.literal_eval`).
> `"42"` → `42`, `"3.14"` → `3.14`, `"[1,2]"` → `[1, 2]`, `"true"` → reste `"true"` (string).

## Flags booléens — syntaxe spécifique Fire

```python
def run(debug: bool = False, verbose: bool = True):
    print(debug, verbose)

fire.Fire(run)
```

```bash
python script.py --debug          # debug=True
python script.py --nodebug        # debug=False
python script.py --noverbose      # verbose=False

# Forme avec valeur explicite (moins idiomatique mais valide)
python script.py --debug=True
python script.py --verbose=False
```

> [!warning] `--no-flag` vs `--noflag`
> Fire utilise `--noflag` (sans tiret), **pas** `--no-flag`.
> `--no-verbose` → erreur ou argument inconnu selon la version.

## Listes et dictionnaires

```python
def train(layers: list, config: dict = None):
    print(layers, config)

fire.Fire(train)
```

```bash
# Listes — notation Python en string
python script.py --layers="[64, 128, 256]"

# Dictionnaires
python script.py --config="{'lr': 0.001, 'epochs': 10}"
```

## Aide — --help

```bash
python script.py --help          # affiche signature + docstring
python script.py subcommand --help   # aide d'une sous-commande
```

```python
def process(n: int, mode: str = "fast"):
    """Traitement principal.

    Args:
        n: Nombre d'éléments à traiter.
        mode: Mode de traitement ('fast' ou 'slow').
    """
    ...

fire.Fire(process)
# python script.py --help → affiche docstring + types + defaults
```

> [!tip] Docstrings → help automatique
> Fire extrait les docstrings des fonctions et classes pour l'aide `--help`.
> Documenter ses fonctions → CLI auto-documentée sans effort supplémentaire.
