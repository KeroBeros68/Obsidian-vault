#python #bases #pièges #erreurs #debugging

## 🪤 Piège 1 — Valeur par défaut mutable

```python
def ajouter(val, liste=[]):   # ❌ la liste est créée une fois pour toutes
    liste.append(val)
    return liste

ajouter(1)   # [1]
ajouter(2)   # [1, 2]  ← pas [2] !

def ajouter(val, liste=None):  # ✅
    if liste is None:
        liste = []
    liste.append(val)
    return liste
```

---

## 🪤 Piège 2 — Copie superficielle vs profonde

```python
original = [[1, 2], [3, 4]]
copie = original[:]       # ❌ copie superficielle — les sous-listes sont partagées
copie[0].append(99)
print(original)            # [[1, 2, 99], [3, 4]] — original modifié !

import copy
copie = copy.deepcopy(original)  # ✅
```

---

## 🪤 Piège 3 — `is` au lieu de `==`

```python
a = 1000
b = 1000
a is b    # False (deux objets distincts)
a == b    # True  ✅

# Fonctionne par coïncidence pour les petits entiers (-5 à 256) mis en cache
x = 5
y = 5
x is y    # True — mais c'est un détail d'implémentation, ne pas s'y fier
```

---

## 🪤 Piège 4 — Modifier une liste en l'itérant

```python
lst = [1, 2, 3, 4, 5]
for x in lst:
    if x % 2 == 0:
        lst.remove(x)    # ❌ saute des éléments

lst = [x for x in lst if x % 2 != 0]  # ✅ nouvelle liste
# ou
for x in lst[:]:   # ✅ itérer sur une copie
    if x % 2 == 0:
        lst.remove(x)
```

---

## 🪤 Piège 5 — `{}` crée un dict, pas un set

```python
vide = {}         # dict ❌ si on voulait un set
vide = set()      # ✅

singleton = {1}   # set — mais {1: ...} serait un dict
```

---

## 🪤 Piège 6 — Portée de variable dans une boucle

```python
fonctions = [lambda: i for i in range(3)]
fonctions[0]()   # 2 — pas 0 !
# Toutes les lambdas capturent la même variable i (valeur finale = 2)

fonctions = [lambda i=i: i for i in range(3)]  # ✅ capturer la valeur
fonctions[0]()   # 0 ✅
```

---

## 🪤 Piège 7 — Comparer des flottants avec `==`

```python
0.1 + 0.2 == 0.3   # False

import math
math.isclose(0.1 + 0.2, 0.3)   # True ✅
```

---

## 🪤 Piège 8 — `except Exception` trop large

```python
try:
    traitement()
except Exception:
    pass   # ❌ masque KeyboardInterrupt, memory errors... et tous les bugs

except Exception as e:
    logger.error(e)   # ✅ au moins logger l'erreur

except ValueError:    # ✅ capturer le type précis
    ...
```

---

## 🪤 Piège 9 — String immuable — concaténation en boucle

```python
resultat = ""
for mot in mots:
    resultat += mot   # ❌ crée une nouvelle chaîne à chaque itération — O(n²)

resultat = "".join(mots)   # ✅ O(n)
```

---

## 🪤 Piège 10 — `range` excluant la borne supérieure

```python
for i in range(5):
    print(i)    # 0 1 2 3 4 — pas 5 !

list(range(1, 6))   # [1, 2, 3, 4, 5]
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Valeur par défaut mutable | Utiliser `None` et initialiser dans le corps |
| Copie superficielle | `copy.deepcopy()` pour les imbriqués |
| `is` vs `==` | `is` uniquement pour `None` et singletons |
| Modifier en itérant | Itérer sur une copie ou utiliser une compréhension |
| `{}` vide | `set()` pour un ensemble vide |
| Lambda en boucle | Capturer la valeur avec `lambda i=i: i` |
| Égalité flottante | `math.isclose()` |
| `except` trop large | Capturer le type précis |
| Concaténation de chaînes | `"".join(liste)` |
| `range` exclusif | `range(1, n+1)` pour inclure `n` |
