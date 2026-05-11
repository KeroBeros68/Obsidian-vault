#python #bases #erreurs #exceptions

## try / except

```python
try:
    resultat = 10 / 0
except ZeroDivisionError:
    print("division par zéro")
```

## Capturer plusieurs exceptions

```python
try:
    val = int(input())
except ValueError:
    print("pas un entier")
except (TypeError, OverflowError):
    print("type ou valeur invalide")
except Exception as e:
    print(f"erreur inattendue : {e}")   # capture tout
```

## else et finally

```python
try:
    f = open("fichier.txt")
except FileNotFoundError:
    print("fichier introuvable")
else:
    contenu = f.read()   # s'exécute si aucune exception
    f.close()
finally:
    print("toujours exécuté")   # nettoyage garanti
```

`finally` s'exécute même si une exception non capturée se propage, ou si un `return` est atteint.

## Lever une exception

```python
def diviser(a, b):
    if b == 0:
        raise ValueError("diviseur ne peut pas être zéro")
    return a / b

raise TypeError("message")           # lever sans contexte
raise                                 # re-propager l'exception courante (dans un except)
```

## Chaîner des exceptions

```python
try:
    int("abc")
except ValueError as e:
    raise RuntimeError("conversion échouée") from e
    # le traceback montre les deux exceptions liées
```

## Hiérarchie des exceptions courantes

```
BaseException
└── Exception
    ├── ValueError      — valeur incorrecte (int("abc"))
    ├── TypeError       — mauvais type d'argument
    ├── KeyError        — clé absente dans un dict
    ├── IndexError      — index hors limites
    ├── AttributeError  — attribut inexistant
    ├── FileNotFoundError (sous-classe d'OSError)
    ├── PermissionError   (sous-classe d'OSError)
    ├── ZeroDivisionError
    ├── StopIteration   — fin d'itérateur
    └── RuntimeError    — erreur générique d'exécution
```

## Exceptions personnalisées

```python
class ErreurMetier(Exception):
    pass

class SoldeInsuffisant(ErreurMetier):
    def __init__(self, montant, solde):
        super().__init__(f"Besoin de {montant}, solde = {solde}")
        self.montant = montant
        self.solde = solde

raise SoldeInsuffisant(100, 30)
```

## Gestionnaire de contexte — with

```python
with open("fichier.txt", "r") as f:
    contenu = f.read()
# f est automatiquement fermé à la sortie du bloc, même en cas d'exception
```

Équivalent à try/finally mais plus concis. Fonctionne avec tout objet implémentant `__enter__` / `__exit__`.

## Ignorer silencieusement

```python
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove("fichier_peut_etre_absent.txt")
# aucune exception si le fichier n'existe pas ✅
```

> [!warning] `except Exception` trop large
> Capturer `Exception` masque des bugs. Capturer le type le plus précis possible. Toujours logger ou afficher `e` si on attrape large.

> [!tip] EAFP plutôt que LBYL
> Le style Python préféré est "Easier to Ask Forgiveness than Permission" : tenter l'opération et gérer l'exception, plutôt que vérifier les conditions à l'avance.
> ```python
> # LBYL (Look Before You Leap) — style C
> if "cle" in d:
>     val = d["cle"]
>
> # EAFP — style Python ✅
> try:
>     val = d["cle"]
> except KeyError:
>     ...
> # ou plus court :
> val = d.get("cle")
> ```
