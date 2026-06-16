#python #fire #cli #fire-flags #interactif #completion #avancé

## Fire flags — options globales après `--`

Les fire flags s'appliquent à Fire lui-même, pas au composant. Ils se placent après `--`.

```bash
python script.py [args composant] -- [fire flags]

python script.py command -- --help          # aide sur le résultat de command
python script.py command -- --interactive   # ouvre un REPL sur le résultat
python script.py command -- --separator=,  # change le séparateur de chaîne
python script.py -- --completion            # génère le script de completion shell
python script.py -- --completion fish       # completion pour fish shell
```

## --help — aide contextuelle

```bash
python script.py --help             # aide sur le composant racine
python script.py sous-cmd --help    # aide sur une sous-commande
python script.py sous-cmd -- --help # aide sur le résultat de sous-cmd
```

```
Exemple de sortie --help :
NAME
    script.py - Calculatrice en ligne de commande.

SYNOPSIS
    script.py COMMAND

COMMANDS
    add
    multiply
```

## --interactive — REPL sur le résultat

```python
import fire

class DataLoader:
    def load(self, path: str):
        import pandas as pd
        return pd.read_csv(path)   # retourne un DataFrame

fire.Fire(DataLoader)
```

```bash
python script.py load data.csv -- --interactive
# Ouvre un shell Python (IPython si disponible)
# avec `result` = le DataFrame chargé
# >>> result.head()
# >>> result.describe()
```

> [!tip] Exploration interactive
> `-- --interactive` est idéal pour inspecter le résultat d'une commande sans écrire de script supplémentaire.
> Pratique pour le debug ou l'exploration de données.

## --completion — autocomplétion shell

```bash
# Bash
python script.py -- --completion bash >> ~/.bashrc
source ~/.bashrc

# Zsh
python script.py -- --completion zsh >> ~/.zshrc

# Fish
python script.py -- --completion fish > ~/.config/fish/completions/script.fish
```

Après installation, `Tab` complète les sous-commandes et les flags.

## @fire.decorators — personnalisation avancée

```python
import fire
import fire.decorators

class MyCLI:
    @fire.decorators.SetParseFn(lambda x: x.split(","))
    def process(self, items):
        """items est parsé depuis une string CSV."""
        print(items)

fire.Fire(MyCLI)
```

```bash
python script.py process "a,b,c"   # → ['a', 'b', 'c']
```

## Tableau récapitulatif des fire flags

| Fire flag | Effet |
|---|---|
| `-- --help` | Aide sur le résultat courant |
| `-- --interactive` | REPL Python/IPython sur le résultat |
| `-- --separator=X` | Change le séparateur de chaîne (défaut : `-`) |
| `-- --completion` | Génère le script de completion (bash/zsh/fish) |
| `-- --trace` | Trace de l'exécution Fire (debug) |

> [!warning] `--` obligatoire pour les fire flags
> `python script.py --interactive` sera interprété comme un argument du composant, pas comme un fire flag.
> Toujours préfixer par `--` : `python script.py -- --interactive`.
