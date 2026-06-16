#python #fire #cli #pièges #erreurs #debugging

## 🪤 Piège 1 — `--noflag` pas `--no-flag`

```bash
python script.py --no-verbose   # ❌ argument non reconnu ou string "--no-verbose"
python script.py --noverbose    # ✅ Fire interprète correctement → verbose=False
```

> [!warning] Différence avec argparse et click
> `argparse` et `click` utilisent `--no-flag`.
> Fire utilise `--noflag` (sans tiret interne).
> Confondre les deux est l'erreur la plus fréquente en migrant depuis argparse.

---

## 🪤 Piège 2 — Fire flag vs argument du composant

```bash
python script.py --interactive   # ❌ interprété comme arg du composant, pas fire flag
python script.py -- --interactive   # ✅ fire flag correctement reconnu
```

```python
# Si le composant a un paramètre "interactive", ambiguïté complète
def run(interactive: bool = False): ...
fire.Fire(run)
# python script.py --interactive → interactive=True du composant ✅
# python script.py -- --interactive → REPL Fire ✅
# Les deux formes font des choses différentes !
```

> [!tip] Règle mémo
> Tout ce qui contrôle Fire (et non ton code) va après `--`.

---

## 🪤 Piège 3 — Coercition de types sans annotation

```python
def process(n, threshold):   # ❌ pas d'annotations
    print(type(n), type(threshold))

fire.Fire(process)
```

```bash
python script.py 10 0.5
# n="10" (str) pas int — Fire utilise ast.literal_eval
# "10" → 10 (int) ← fonctionne
# "0.5" → 0.5 (float) ← fonctionne
# "true" → "true" (str, pas bool) ← surprenant !
```

```python
# ✅ Toujours annoter les paramètres pour un comportement prévisible
def process(n: int, threshold: float): ...
```

> [!warning] `"true"` sans annotation → reste une string
> Sans annotation `bool`, Fire ne convertit pas `"true"` en `True`.
> Annoter `verbose: bool` pour que `--verbose` et `--noverbose` fonctionnent correctement.

---

## 🪤 Piège 4 — Retour de valeur affiché en sortie

```python
def compute(x: int, y: int) -> int:
    return x + y   # ⚠️ Fire affiche "3" sur stdout

fire.Fire(compute)
```

```bash
python script.py 1 2
# 3    ← affiché automatiquement par Fire
```

```python
# Si l'affichage automatique pose problème (pipelines, tests) :
def compute(x: int, y: int) -> None:
    print(x + y)   # ✅ contrôle explicite de stdout
    # ou stocker dans un fichier / logger
```

> [!tip] Comportement voulu vs subi
> Fire affiche toute valeur non-None retournée.
> Utiliser `return None` ou `print()` explicitement selon l'intention.

---

## 🪤 Piège 5 — Conflits de noms avec les fire flags

```python
def run(help: str = "default"):   # ❌ `help` est aussi un fire flag
    print(help)

fire.Fire(run)
```

```bash
python script.py --help   # → affiche l'aide Fire, pas help="aide" du composant
```

```python
# ✅ Renommer le paramètre pour éviter le conflit
def run(description: str = "default"):
    print(description)
```

> [!warning] Noms réservés à éviter comme paramètres
> `help`, `interactive`, `separator`, `completion`, `trace` peuvent entrer en conflit avec les fire flags.

---

## 🪤 Piège 6 — `__init__` avec des types non primitifs

```python
class Pipeline:
    def __init__(self, config: dict):   # ❌ difficile à passer en CLI
        self.config = config

fire.Fire(Pipeline)
```

```bash
python script.py --config="{'lr': 0.001}"   # fragile : guillemets, espaces
```

```python
# ✅ Passer un chemin de fichier, charger en interne
class Pipeline:
    def __init__(self, config_path: str = "config.yaml"):
        import yaml
        with open(config_path) as f:
            self.config = yaml.safe_load(f)
```

> [!tip] Règle générale
> Les `__init__` Fire doivent accepter uniquement des types primitifs (str, int, float, bool)
> ou des listes/dicts simples. Charger les objets complexes dans le corps de `__init__`.

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| `--no-flag` ne fonctionne pas | Utiliser `--noflag` (sans tiret) |
| `--interactive` non reconnu comme fire flag | Placer après `--` : `-- --interactive` |
| Type non converti sans annotation | Annoter tous les paramètres |
| Retour affiché indésirable | Retourner `None` ou utiliser `print()` |
| Conflit avec fire flags (`help`, `trace`) | Renommer les paramètres du composant |
| `__init__` avec types complexes | Utiliser des chemins de fichiers + chargement interne |
