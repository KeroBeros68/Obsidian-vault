#python #bases #variables #types

## Assignation

```python
x = 42
nom = "Alice"
pi = 3.14
actif = True
rien = None
```

Pas de déclaration de type obligatoire. Le type est déterminé à l'exécution.

## Types primitifs

| Type | Exemple | Remarque |
|------|---------|----------|
| `int` | `42`, `-7`, `0xFF`, `0b1010` | Précision arbitraire |
| `float` | `3.14`, `1e-3`, `float('inf')` | IEEE 754 double précision |
| `bool` | `True`, `False` | Sous-classe de `int` : `True == 1` |
| `str` | `"hello"`, `'world'` | Immuable, Unicode |
| `NoneType` | `None` | Valeur unique de son type |
| `complex` | `3+2j` | Partie réelle + imaginaire |

## Vérifier et convertir un type

```python
type(42)          # <class 'int'>
isinstance(42, int)  # True

int("42")         # 42   — str → int
float(3)          # 3.0  — int → float
str(42)           # "42" — int → str
bool(0)           # False — 0, "", [], None sont falsy
```

## Assignation multiple

```python
a = b = c = 0           # tous à 0
x, y = 10, 20           # déballage (unpacking)
a, b = b, a             # échange sans variable temporaire ✅
x, *reste = [1, 2, 3, 4]  # x=1, reste=[2, 3, 4]
```

## Chaînes de caractères

```python
s = "bonjour"
s[0]           # "b" — indexation
s[-1]          # "r" — depuis la fin
s[1:4]         # "onj" — slicing
len(s)         # 7
s.upper()      # "BONJOUR"
s.lower()      # "bonjour"
s.strip()      # supprime espaces/newlines en début et fin
s.replace("o", "0")  # "b0nj0ur"
s.split("n")   # ["bo", "jour"]
"jour" in s    # True
```

## f-strings (Python 3.6+)

```python
nom = "Alice"
age = 30

f"Bonjour {nom}"            # "Bonjour Alice"
f"Dans 10 ans : {age + 10}" # "Dans 10 ans : 40"
f"{pi:.2f}"                 # "3.14" — formatage
f"{1000000:_}"              # "1_000_000" — séparateur milliers
f"{valeur!r}"               # repr() de la valeur
```

## Nombres

```python
10 / 3     # 3.3333... — division flottante
10 // 3    # 3         — division entière
10 % 3     # 1         — modulo
2 ** 8     # 256       — puissance
abs(-5)    # 5
round(3.14159, 2)  # 3.14

# Bases
0xFF       # 255 (hex)
0b1010     # 10  (binaire)
0o17       # 15  (octal)
```

> [!warning] Égalité flottante
> ```python
> 0.1 + 0.2 == 0.3   # False — imprécision IEEE 754
> import math
> math.isclose(0.1 + 0.2, 0.3)  # True ✅
> ```
