#python #bases #listes #tuples

## Listes

```python
l = [1, 2, 3]
l = [1, "deux", 3.0, True]   # types hétérogènes
l = []                        # liste vide
l = list(range(5))            # [0, 1, 2, 3, 4]
```

## Indexation & slicing

```python
l = [10, 20, 30, 40, 50]

l[0]      # 10  — premier
l[-1]     # 50  — dernier
l[1:3]    # [20, 30]  — de l'index 1 (inclus) à 3 (exclu)
l[:2]     # [10, 20]
l[2:]     # [30, 40, 50]
l[::2]    # [10, 30, 50] — un élément sur deux
l[::-1]   # [50, 40, 30, 20, 10] — inversé
```

Le slicing retourne toujours une **copie**.

## Méthodes courantes

```python
l.append(6)          # ajoute à la fin — O(1)
l.insert(0, 99)      # insère à l'index 0 — O(n)
l.extend([7, 8])     # concatène une autre liste en place
l.pop()              # retire et retourne le dernier — O(1)
l.pop(0)             # retire et retourne l'index 0 — O(n)
l.remove(30)         # retire la première occurrence de 30
l.index(20)          # index de la première occurrence de 20
l.count(1)           # nombre d'occurrences de 1
l.reverse()          # inverse en place
l.sort()             # trie en place (croissant)
l.sort(reverse=True) # trie en place (décroissant)
l.clear()            # vide la liste
len(l)               # nombre d'éléments
```

## Opérations

```python
[1, 2] + [3, 4]    # [1, 2, 3, 4] — concaténation (nouvelle liste)
[0] * 5            # [0, 0, 0, 0, 0] — répétition
3 in [1, 2, 3]     # True
sorted([3,1,2])    # [1, 2, 3] — nouvelle liste triée
min([3,1,2])       # 1
max([3,1,2])       # 3
sum([1,2,3])       # 6
```

## Copie de liste

```python
original = [1, 2, 3]

ref = original        # ❌ même objet — modifications partagées
copie = original[:]   # ✅ copie superficielle
copie = list(original) # ✅ idem
copie = original.copy()# ✅ idem

import copy
profonde = copy.deepcopy(original)  # copie profonde — pour les imbriqués
```

## Tuples

```python
t = (1, 2, 3)
t = 1, 2, 3         # les parenthèses sont optionnelles
t = (42,)           # tuple à un seul élément — la virgule est obligatoire
t = ()              # tuple vide

t[0]                # 1 — indexation identique aux listes
t[1:]               # (2, 3)
len(t)              # 3
```

Un tuple est **immuable** : on ne peut pas modifier, ajouter ou supprimer d'éléments après création.

## Tuples vs Listes

| | Liste | Tuple |
|--|-------|-------|
| Syntaxe | `[1, 2, 3]` | `(1, 2, 3)` |
| Mutable | ✅ | ❌ |
| Hashable | ❌ | ✅ (si contenu hashable) |
| Usage typique | Collection évolutive | Données fixes, clé de dict |

## Déballage (unpacking)

```python
coords = (48.8, 2.3)
lat, lon = coords           # déballage en 2 variables

premier, *reste = [1,2,3,4] # premier=1, reste=[2,3,4]
*debut, dernier = [1,2,3,4] # debut=[1,2,3], dernier=4
a, _, c = (1, 2, 3)         # _ pour ignorer une valeur
```

> [!tip] Préférer les tuples pour les données constantes
> Un tuple est plus rapide qu'une liste à créer et consomme moins de mémoire. Il communique l'intention : "ces données ne changent pas."
