#python #numpy #broadcasting #maths #ufunc

## Opérations élément par élément

```python
a + b    # addition
a * b    # multiplication
a ** 2   # puissance
a / 2    # division
a % 3    # modulo
```

## Ufuncs essentielles

```python
np.sqrt(a)      # racine carrée
np.exp(a)       # exponentielle
np.log(a)       # logarithme naturel
np.abs(a)       # valeur absolue
np.sin(a)       # sinus
np.round(a, 2)  # arrondi
```

## Broadcasting — règles

NumPy compare les shapes **de droite à gauche**. Deux dimensions sont compatibles si :

- elles sont **égales**, ou
- l'une d'elles vaut **1** (elle est étirée)

```
(2, 3) + (3,)    → (2, 3)  ✅
(2, 3) + (2, 1)  → (2, 3)  ✅
(5, 3, 1) + (3, 4) → (5, 3, 4) ✅
(2, 3) + (2, 2)  → ERREUR   ❌
```

> [!tip] Le `1` est flexible Une dimension qui vaut `1` est **toujours** compatible — elle s'étire pour correspondre à l'autre.

## Comparaisons → array booléen

```python
a > 3        # [False True ...]
a == 5       # [False True ...]
a != 2       # [True  True ...]
```
