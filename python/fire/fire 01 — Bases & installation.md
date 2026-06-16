#python #fire #cli #bases #installation

## Installation

```bash
pip install fire
```

## Concept — zéro boilerplate

Fire inspecte un objet Python (fonction, classe, instance, dict, module) et génère automatiquement un CLI complet à partir de sa signature.

```python
# Sans Fire — argparse minimal
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--name")
args = parser.parse_args()
print(f"Hello {args.name}")

# Avec Fire — même résultat
import fire
def greet(name): print(f"Hello {name}")
fire.Fire(greet)
# python script.py --name Kevin  →  Hello Kevin
```

## fire.Fire() avec une fonction

```python
import fire

def add(x: int, y: int) -> int:
    """Additionne deux entiers."""
    return x + y

fire.Fire(add)
```

```bash
python script.py 3 5        # → 8  (positionnels)
python script.py --x=3 --y=5  # → 8  (nommés)
python script.py --help     # → affiche la signature + docstring
```

## fire.Fire() avec une classe

```python
import fire

class Calculator:
    """Calculatrice en ligne de commande."""

    def add(self, x: int, y: int) -> int:
        return x + y

    def multiply(self, x: int, y: int) -> int:
        return x * y

fire.Fire(Calculator)
```

```bash
python script.py add 3 5       # → 8
python script.py multiply 3 5  # → 15
python script.py --help        # → liste les méthodes disponibles
```

## fire.Fire() avec un dict

```python
import fire

def train(epochs: int = 10): print(f"Training {epochs} epochs")
def evaluate(split: str = "test"): print(f"Evaluating on {split}")

fire.Fire({"train": train, "eval": evaluate})
```

```bash
python script.py train --epochs=20   # → Training 20 epochs
python script.py eval --split=val    # → Evaluating on val
```

## fire.Fire() sans argument — expose le module courant

```python
import fire

def hello(name: str = "World"):
    return f"Hello {name}!"

def goodbye(name: str = "World"):
    return f"Goodbye {name}!"

if __name__ == "__main__":
    fire.Fire()   # expose toutes les fonctions du module courant
```

```bash
python script.py hello --name=Kevin   # → Hello Kevin!
python script.py goodbye              # → Goodbye World!
```

> [!tip] Quel mode choisir ?
> Fonction unique → `fire.Fire(fn)`.
> Plusieurs sous-commandes liées → `fire.Fire(MyClass)`.
> Sous-commandes indépendantes → `fire.Fire({"cmd1": fn1, "cmd2": fn2})`.
> Exposer tout le module → `fire.Fire()`.

> [!info] Retour de valeur
> Fire affiche le retour de la fonction si ce n'est pas `None`.
> Pour supprimer l'affichage, retourner `None` ou utiliser `print()` explicitement.
