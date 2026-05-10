#python #numpy #booléen #filtrage #masque #where

## Masque booléen

```python
a[a > 4]           # filtre les éléments > 4
masque = a > 4     # crée le masque séparément
a[masque]          # applique le masque
```

## Combiner des conditions

```python
a[(a > 2) & (a < 7)]   # ET  — entre 2 et 7
a[(a < 2) | (a > 6)]   # OU  — inférieur à 2 ou supérieur à 6
a[~(a > 4)]            # NON — tout ce qui n'est pas > 4
```

> [!warning] Parenthèses obligatoires ! `a[(a > 2) & (a < 7)]` ✅ `a[a > 2 & a < 7]` ❌ — erreur de priorité des opérateurs

## np.where — if/else vectorisé

```python
np.where(condition, valeur_si_vrai, valeur_si_faux)

np.where(a > 4, 1, 0)      # binaire
np.where(a > 4, a, -1)     # garde la valeur ou remplace
```

## Fonctions booléennes

```python
np.any(a > 6)       # au moins un élément vérifie ?
np.all(a > 0)       # tous les éléments vérifient ?
np.sum(a > 4)       # combien d'éléments vérifient ?
np.mean(a > 4)      # proportion qui vérifie (0.0 à 1.0)
np.argwhere(a > 4)  # indices des éléments qui vérifient
```

## Assignation avec masque

```python
a[a > 6] = 0           # remplace les valeurs > 6 par 0
a[a % 2 == 0] *= -1    # modifie les pairs
```
