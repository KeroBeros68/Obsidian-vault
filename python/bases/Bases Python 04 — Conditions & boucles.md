#python #bases #conditions #boucles

## if / elif / else

```python
x = 10

if x > 0:
    print("positif")
elif x < 0:
    print("négatif")
else:
    print("zéro")
```

L'indentation (4 espaces) définit les blocs — pas d'accolades.

## Valeurs falsy

Évaluées à `False` dans un contexte booléen :

```python
False, None, 0, 0.0, 0j    # valeurs numériques nulles
"", [], {}, set(), ()       # conteneurs vides
```

Tout le reste est truthy.

```python
if liste:          # ✅ équivalent à if len(liste) > 0
if not dictionnaire:   # ✅ si vide
```

## Opérateurs de comparaison

```python
x == y    # égalité de valeur
x != y    # différence
x is y    # même objet en mémoire (identité)
x is not y
x in collection    # appartenance
x not in collection
```

> [!warning] `is` vs `==`
> ```python
> a = [1, 2]
> b = [1, 2]
> a == b    # True  — mêmes valeurs
> a is b    # False — objets distincts
>
> # Utiliser is uniquement pour None et les singletons
> if x is None:  # ✅
> if x == None:  # ❌ déconseillé
> ```

## Expression conditionnelle (ternaire)

```python
label = "pair" if n % 2 == 0 else "impair"
```

## for — itération

```python
for x in [1, 2, 3]:
    print(x)

for c in "bonjour":   # itère sur les caractères
    print(c)

for i in range(5):      # 0 1 2 3 4
for i in range(2, 8):   # 2 3 4 5 6 7
for i in range(0, 10, 2): # 0 2 4 6 8 — avec pas
```

## Fonctions d'itération utiles

```python
for i, val in enumerate(["a","b","c"]):
    print(i, val)      # 0 a / 1 b / 2 c

for a, b in zip([1,2,3], ["x","y","z"]):
    print(a, b)        # 1 x / 2 y / 3 z

for val in reversed([1,2,3]):   # 3 2 1
for val in sorted([3,1,2]):     # 1 2 3
```

## while

```python
n = 10
while n > 0:
    print(n)
    n -= 1

# Boucle infinie avec sortie explicite
while True:
    reponse = input("> ")
    if reponse == "quit":
        break
```

## break / continue / else

```python
for i in range(10):
    if i == 3:
        continue   # passe à l'itération suivante
    if i == 7:
        break      # sort de la boucle
    print(i)

# else sur une boucle — s'exécute si la boucle s'est terminée sans break
for i in range(5):
    if i == 10:
        break
else:
    print("aucun break — boucle complète")  # s'affiche ici
```

## Itération sur un dict

```python
d = {"a": 1, "b": 2, "c": 3}

for cle in d:              # clés
for val in d.values():     # valeurs
for cle, val in d.items(): # paires ✅

# Ne pas modifier un dict pendant qu'on l'itère
for k in list(d.keys()):   # copier les clés d'abord ✅
    if condition:
        del d[k]
```

> [!tip] `enumerate` plutôt qu'un compteur manuel
> ```python
> # ❌ style C
> i = 0
> for val in liste:
>     print(i, val)
>     i += 1
>
> # ✅ Pythonic
> for i, val in enumerate(liste):
>     print(i, val)
> ```
