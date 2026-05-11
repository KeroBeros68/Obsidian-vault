#python #bases #dictionnaires #ensembles

## Dictionnaires

```python
d = {"nom": "Alice", "age": 30}
d = {}                          # dict vide
d = dict(nom="Alice", age=30)   # syntaxe alternative
```

Les clés doivent être **hashables** (str, int, tuple, ...). Les valeurs peuvent être de n'importe quel type.

## Accès & modification

```python
d["nom"]            # "Alice" — KeyError si clé absente
d.get("age")        # 30
d.get("ville", "?") # "?" si clé absente (valeur par défaut) ✅

d["ville"] = "Paris"  # ajout ou modification
del d["age"]          # suppression — KeyError si absente
d.pop("nom")          # supprime et retourne la valeur
d.pop("x", None)      # sans KeyError si absente ✅

"nom" in d          # True — test d'appartenance
len(d)              # nombre de paires
```

## Itération

```python
for cle in d:              # itère sur les clés
for cle in d.keys():       # idem, explicite
for val in d.values():     # itère sur les valeurs
for cle, val in d.items(): # itère sur les paires ✅ (le plus courant)
    print(f"{cle} → {val}")
```

## Méthodes utiles

```python
d.update({"x": 1, "y": 2})    # fusionne (modifie d en place)
d | {"x": 1}                   # fusion — nouvelle dict (Python 3.9+)
d |= {"x": 1}                  # fusion en place (Python 3.9+)

d.setdefault("score", 0)       # ajoute "score":0 seulement si absent
d.copy()                       # copie superficielle
```

## Dict par compréhension

```python
carrés = {n: n**2 for n in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

inversé = {v: k for k, v in d.items()}
```

## Ensembles (set)

```python
s = {1, 2, 3}
s = set()           # ensemble vide — {} crée un dict !
s = set([1,1,2,3])  # {1, 2, 3} — doublons supprimés
```

Les éléments doivent être **hashables**. Un ensemble est **non ordonné** (pas d'indexation).

## Opérations ensemblistes

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b    # union          {1, 2, 3, 4, 5, 6}
a & b    # intersection   {3, 4}
a - b    # différence     {1, 2}
a ^ b    # différence sym {1, 2, 5, 6}

a.issubset(b)    # a ⊆ b ?
a.issuperset(b)  # a ⊇ b ?
a.isdisjoint(b)  # aucun élément commun ?
```

## Méthodes set

```python
s.add(4)          # ajoute un élément
s.remove(4)       # supprime — KeyError si absent
s.discard(4)      # supprime sans erreur si absent ✅
s.pop()           # retire un élément arbitraire
s.clear()         # vide l'ensemble
4 in s            # test d'appartenance — O(1)
```

## Cas d'usage

| Structure | Quand l'utiliser |
|-----------|----------------|
| `dict` | Associer des clés à des valeurs, lookup par clé |
| `set` | Dédoublonner, tester l'appartenance rapidement, opérations ensemblistes |

```python
# Dédoublonner une liste (ordre non conservé)
unique = list(set([3, 1, 2, 1, 3]))

# Lookup rapide
valides = {"admin", "moderateur", "editeur"}
if role in valides:   # O(1) vs O(n) pour une liste ✅
    ...
```

> [!warning] `{}` crée un dict, pas un set
> ```python
> vide_dict = {}        # dict ❌ si on voulait un set
> vide_set  = set()     # ✅
> ```
