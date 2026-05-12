#python #generateurs #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **itérable** | Objet implémentant `__iter__()`. Peut produire un itérateur. Ex : `list`, `str`, `dict`, `range`. |
| **itérateur** | Objet implémentant `__iter__()` et `__next__()`. Produit les valeurs une à une. Épuisable. |
| **protocole itérateur** | Contrat Python : `__iter__` retourne `self`, `__next__` retourne la valeur suivante ou lève `StopIteration`. |
| **`StopIteration`** | Exception levée par `__next__` quand il n'y a plus de valeurs. Signal de fin d'itération pour `for`. |
| **`iter(obj)`** | Appelle `obj.__iter__()`. Retourne l'itérateur associé. |
| **`next(it)`** | Appelle `it.__next__()`. Retourne la valeur suivante ou lève `StopIteration`. |
| **`next(it, défaut)`** | Retourne `défaut` au lieu de lever `StopIteration`. |
| **générateur** | Itérateur créé par une fonction génératrice ou une expression génératrice. Usage unique. |
| **fonction génératrice** | Fonction contenant `yield`. Retourne un objet générateur à l'appel. |
| **`yield`** | Suspend l'exécution de la fonction, retourne une valeur à l'appelant, reprend au prochain `next()`. |
| **`yield from`** | Délègue l'itération à un sous-itérable. Propage `send()`, `throw()`, et la valeur de `return`. |
| **évaluation paresseuse** | Les valeurs sont calculées à la demande, pas toutes d'un coup. Économise la mémoire. |
| **expression génératrice** | `(expr for x in it)` — syntaxe compacte créant un générateur sans `def`. |
| **amorçage** | Premier appel `next(gen)` ou `gen.send(None)` qui avance jusqu'au premier `yield`. Obligatoire avant `send(valeur)`. |
| **`send(val)`** | Injecte `val` dans le générateur suspendu au `yield`. `yield` s'évalue alors à `val`. |
| **`throw(exc)`** | Lève `exc` à l'endroit où le générateur est suspendu. |
| **`close()`** | Lève `GeneratorExit` au `yield` courant. Permet au générateur de se nettoyer via `finally`. |
| **`GeneratorExit`** | Exception spéciale levée par `close()`. Ne doit pas être ignorée. |
| **itérateur infini** | Générateur sans `StopIteration`. À utiliser avec `islice` ou `takewhile` pour limiter. |
| **pipeline** | Chaîne de générateurs où chaque étape consomme la précédente paresseusement. |
| **`itertools`** | Module standard fournissant des générateurs efficaces : `chain`, `islice`, `product`, `groupby`... |
