#numpy #pièges #erreurs #debugging

## 🪤 Piège 1 — stop toujours exclu

`arange`, `linspace` et le slicing excluent tous la borne `stop`.

python

```python
np.arange(1, 10, 3)   # [1 4 7]  — le 10 est EXCLU
a[1:4]                # indices 1, 2, 3  — le 4 est EXCLU
```

> [!warning] `np.arange(1, 10, 3)` → `[1 4 7]` et NON `[1 4 7 10]`

---

## 🪤 Piège 2 — slice retourne une vue, pas une copie

python

```python
a = np.array([1, 2, 3, 4, 5])
b = a[1:4]    # b est une VUE sur a
b[0] = 99
print(a)      # [1 99 3 4 5]  ← a est modifié !

# Solution :
b = a[1:4].copy()
```

> [!warning] Si tu veux travailler sur une portion sans toucher l'original → toujours `.copy()`

---

## 🪤 Piège 3 — axis=0 vs axis=1 inversé

L'erreur la plus fréquente sur les agrégations 2D.

python

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

np.sum(m, axis=0)   # [5 7 9]  — somme par COLONNE (descend ↓)
np.sum(m, axis=1)   # [6 15]   — somme par LIGNE   (traverse →)
```

> [!tip] Mémo `axis=0` **efface** les lignes → il reste les colonnes `axis=1` **efface** les colonnes → il reste les lignes

---

## 🪤 Piège 4 — `*` vs `@` pour les matrices

python

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[2, 0], [1, 3]])

a * b   # [[2, 0], [3, 12]]   ← élément par élément
a @ b   # [[4, 6], [10, 12]]  ← produit MATRICIEL
```

> [!warning] Les deux sont syntaxiquement valides — NumPy ne lève pas d'erreur. Le bug est silencieux et peut fausser tout un modèle ML !

---

## 🪤 Piège 5 — upcasting silencieux

python

```python
a = np.array([1, 2, 3])         # int64
b = np.array([1, 2, 3.0])       # float64 — le 3.0 "contamine" tout
c = np.array([1, 2, 3], dtype=np.int32)
d = (c + 1.5)                   # → float64, plus int32 !
```

> [!tip] Toujours vérifier `a.dtype` après une opération si le type est important pour toi (mémoire, précision).

---

## 🪤 Piège 6 — parenthèses obligatoires dans les masques

python

```python
a[(a > 2) & (a < 7)]   # ✅ correct
a[a > 2 & a < 7]       # ❌ erreur — & a une priorité plus haute que >
```

> [!warning] Sans parenthèses, Python évalue `2 & a` avant la comparaison → résultat inattendu ou erreur.

---

## 🪤 Piège 7 — reshape échoue si le total ne correspond pas

python

```python
a = np.arange(12)
a.reshape(3, 5)   # ❌ ValueError : 3×5=15 ≠ 12
a.reshape(3, 4)   # ✅ 3×4=12
a.reshape(3, -1)  # ✅ NumPy calcule : 12/3 = 4
```

> [!tip] Utilise `-1` pour laisser NumPy calculer la dimension manquante.

---

## 🪤 Piège 8 — broadcasting : le `1` est compatible, pas les autres

python

```python
# shape (2, 3) + shape (2, 1) → ✅  le 1 est étiré à 3
# shape (2, 3) + shape (2, 2) → ❌  2 ≠ 3, pas de règle qui sauve

# Piège subtil :
# shape (5, 3, 1) + shape (3, 4) → ✅  résultat (5, 3, 4)
# Le 1 s'étire à 4, et le (3,4) est complété en (1,3,4) puis (5,3,4)
```

> [!warning] L'instinct dit "1 et 4 c'est différent → erreur" mais c'est l'inverse : **le `1` est la valeur la plus flexible du broadcasting.**

---

## 🪤 Piège 9 — `np.sum` sur un booléen compte les True

python

```python
a = np.array([1, 5, 3, 8, 2])
np.sum(a > 3)    # 2  ← compte les éléments > 3 (True=1, False=0)
np.mean(a > 3)   # 0.4 ← proportion (40%)
```

C'est un **pattern voulu**, pas un bug — mais surprenant si on ne le connaît pas.

---

## 🪤 Piège 10 — fancy indexing retourne une copie

python

```python
a = np.array([10, 20, 30, 40])
b = a[[0, 2]]    # fancy indexing → COPIE
b[0] = 99
print(a)         # [10 20 30 40]  ← a inchangé

# Alors que :
c = a[0:2]       # slice → VUE
c[0] = 99
print(a)         # [99 20 30 40]  ← a modifié !
```

> [!tip] Règle simple `[]` avec des **entiers/slices** → vue possible `[]` avec une **liste d'indices** → toujours une copie

---

## Récapitulatif rapide

| Piège                        | Solution                                          |
| ---------------------------- | ------------------------------------------------- |
| stop exclu dans arange/slice | Toujours exclure mentalement la borne stop        |
| Slice = vue                  | Ajouter `.copy()` si indépendance voulue          |
| axis=0 vs axis=1             | axis=0 → résultat par colonne, axis=1 → par ligne |
| `*` vs `@`                   | `*` = élément par élément, `@` = matriciel        |
| Upcasting silencieux         | Vérifier `.dtype` après les opérations            |
| Parenthèses masques          | Toujours entourer chaque condition de `()`        |
| reshape impossible           | Vérifier que m×n = size, ou utiliser `-1`         |
| Broadcasting avec `1`        | Le `1` s'étire toujours — c'est voulu             |
| sum sur booléen              | C'est un pattern utile (compte les True)          |
| Fancy indexing = copie       | slice → vue, liste d'indices → copie              |