#python #fire #cli #classe #oop

## Principe — classe = CLI structurée

```
__init__  → arguments globaux (options partagées par toutes les sous-commandes)
méthodes  → sous-commandes
```

C'est le pattern le plus puissant de Fire pour des CLIs avec plusieurs commandes liées.

## Exemple complet

```python
import fire

class Pipeline:
    """Pipeline de traitement de données."""

    def __init__(self, data_path: str, verbose: bool = False):
        self.data_path = data_path
        self.verbose   = verbose
        if verbose: print(f"[init] data_path={data_path}")

    def train(self, epochs: int = 10, lr: float = 0.001):
        """Lance l'entraînement."""
        print(f"Training {epochs} epochs | lr={lr} | data={self.data_path}")

    def evaluate(self, split: str = "test"):
        """Évalue le modèle."""
        print(f"Evaluating on {split} | data={self.data_path}")

    def export(self, output: str = "model.pkl"):
        """Exporte le modèle entraîné."""
        print(f"Exporting to {output}")

fire.Fire(Pipeline)
```

```bash
# Format général : python script.py [__init__ args] sous-commande [args]
python script.py --data_path=data.csv train --epochs=20
python script.py --data_path=data.csv --verbose train
python script.py --data_path=data.csv evaluate --split=val
python script.py --help          # liste __init__ args + méthodes
python script.py train --help    # détail de la sous-commande train
```

## Méthodes privées — exclues automatiquement

```python
class MyCLI:
    def public_cmd(self):    ...   # ✅ exposée
    def _private_cmd(self):  ...   # ✅ non exposée (underscore)
    def __dunder__(self):    ...   # ✅ non exposée
```

## Instance existante — fire.Fire(obj)

```python
class Greeter:
    def __init__(self, name: str):
        self.name = name

    def hello(self): return f"Hello {self.name}!"
    def bye(self):   return f"Bye {self.name}!"

obj = Greeter("Kevin")   # instance préconstruite
fire.Fire(obj)           # --name n'est plus un arg CLI
```

```bash
python script.py hello   # → Hello Kevin!
python script.py bye     # → Bye Kevin!
```

## Héritage — méthodes héritées exposées

```python
class Base:
    def status(self): return "OK"

class MyCLI(Base):
    def run(self): return "running"

fire.Fire(MyCLI)
# python script.py status  → OK   ✅ méthode héritée exposée
# python script.py run     → running
```

> [!tip] Quand utiliser une classe vs un dict
> Classe : sous-commandes liées qui partagent un état (`self.data_path`, `self.config`).
> Dict : sous-commandes indépendantes sans état partagé.

> [!warning] `__init__` avec des types complexes
> Les arguments de `__init__` doivent être coercibles depuis la CLI (types primitifs ou listes).
> Passer un objet complexe comme `config: MyConfig` n'est pas directement supporté — utiliser un chemin de fichier + chargement interne.
