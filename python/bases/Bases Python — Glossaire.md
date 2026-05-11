#python #bases #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **typage dynamique** | Le type d'une variable est déterminé à l'exécution, pas à la déclaration. |
| **typage fort** | Python refuse les opérations entre types incompatibles : `"3" + 3` → `TypeError`. |
| **falsy** | Valeur évaluée à `False` dans un contexte booléen : `0`, `""`, `[]`, `{}`, `None`, `False`. |
| **truthy** | Toute valeur non falsy. |
| **immuable** | Objet dont la valeur ne peut pas changer après création : `int`, `float`, `str`, `tuple`, `frozenset`. |
| **mutable** | Objet modifiable en place : `list`, `dict`, `set`. |
| **hashable** | Objet pouvant servir de clé de dict ou d'élément de set. Doit être immuable et implémenter `__hash__`. |
| **déballage (unpacking)** | Assignation simultanée depuis un itérable : `a, b = (1, 2)`. |
| **itérable** | Objet qu'on peut parcourir avec `for` : listes, tuples, chaînes, dict, generators... |
| **itérateur** | Objet avec `__next__()` retournant le prochain élément. Épuisable. |
| **générateur** | Fonction avec `yield` — produit les valeurs à la demande. Économise la mémoire. |
| **expression génératrice** | `(x for x in ...)` — compréhension paresseuse, retourne un générateur. |
| **compréhension de liste** | `[expr for x in iterable if cond]` — syntaxe concise pour construire une liste. |
| **portée (scope)** | Zone du code où un nom est visible. Python suit la règle LEGB. |
| **LEGB** | Ordre de résolution des noms : Local → Enclosing → Global → Built-in. |
| **`global`** | Déclare dans une fonction qu'on veut accéder à la variable globale et la modifier. |
| **`nonlocal`** | Déclare qu'on veut accéder à la variable de la fonction englobante. |
| **`*args`** | Paramètre recevant les arguments positionnels supplémentaires sous forme de tuple. |
| **`**kwargs`** | Paramètre recevant les arguments nommés supplémentaires sous forme de dict. |
| **exception** | Objet signalant une erreur à l'exécution. Se propage jusqu'au premier `except` compatible. |
| **`raise`** | Lève une exception manuellement. |
| **gestionnaire de contexte** | Objet utilisé avec `with` — garantit l'exécution de code de nettoyage (`__enter__`/`__exit__`). |
| **module** | Fichier `.py` importable. |
| **package** | Dossier contenant un `__init__.py` — regroupe des modules. |
| **`__name__`** | `"__main__"` si le fichier est exécuté directement, sinon le nom du module. |
| **f-string** | Chaîne préfixée `f"..."` permettant d'insérer des expressions Python via `{...}` (Python 3.6+). |
| **slicing** | Extraction d'une sous-séquence : `s[start:stop:step]`. Retourne une copie. |
