#python #fire #cli #composants #chaînage #avancé

## Composants imbriqués — groupes de sous-commandes

Un composant peut contenir d'autres composants (classes, dicts, fonctions) pour créer une hiérarchie de sous-commandes.

```python
import fire

class DataCommands:
    def download(self, url: str): print(f"Downloading {url}")
    def clean(self, path: str):   print(f"Cleaning {path}")

class ModelCommands:
    def train(self, epochs: int = 10): print(f"Training {epochs} epochs")
    def evaluate(self, split: str = "test"): print(f"Evaluating {split}")

class CLI:
    def __init__(self):
        self.data  = DataCommands()
        self.model = ModelCommands()

fire.Fire(CLI)
```

```bash
python script.py data download --url=http://...
python script.py data clean --path=data/
python script.py model train --epochs=20
python script.py model evaluate
python script.py model --help   # aide du groupe model
```

## Composants via dict imbriqué

```python
fire.Fire({
    "data":  {"download": download_fn, "clean": clean_fn},
    "model": {"train": train_fn, "eval": eval_fn},
})
```

```bash
python script.py data download --url=...
python script.py model train
```

## Chaînage de méthodes

Fire supporte le chaînage : chaque appel de méthode reçoit le résultat de l'appel précédent.  
Le séparateur de chaîne est `-` (tiret simple) par défaut.

```python
import fire

class Builder:
    def __init__(self):
        self.steps = []

    def add(self, step: str):
        self.steps.append(step)
        return self   # ← retourner self pour le chaînage

    def run(self):
        print(f"Running: {self.steps}")

fire.Fire(Builder)
```

```bash
# Chaîner avec le séparateur par défaut `-`
python script.py add preprocess - add train - add evaluate - run
# → Running: ['preprocess', 'train', 'evaluate']
```

## Séparateur de chaîne — changer le défaut

```bash
# Changer le séparateur via -- --separator
python script.py add preprocess , add train , run -- --separator=,

# Utile quand les arguments contiennent des tirets
python script.py --input=my-file - process    # ambigu avec - par défaut
python script.py --input=my-file + process -- --separator=+   # ✅ clair
```

## Séparateur `--` — fire flags vs args du composant

```
python script.py [args du composant] -- [fire flags]

Les deux séparateurs ont des rôles distincts :
  -  (tiret simple)   → sépare les appels chainés sur le résultat
  -- (double tiret)   → sépare les args du composant des options Fire globales
```

```bash
python script.py command arg -- --help          # aide sur le résultat de command
python script.py command1 arg - command2 arg2   # chaînage
```

> [!tip] Fluent API
> Retourner `self` dans chaque méthode active le chaînage Fire.
> Pattern naturel pour les builders, pipelines de traitement, data preprocessing.

> [!warning] Chaînage et types de retour
> Si une méthode retourne un type non-Fire-compatible (ex : un DataFrame Pandas),
> Fire tentera de l'afficher brut. Terminer la chaîne par une méthode qui retourne
> `None` ou une valeur primitive pour un affichage propre.
