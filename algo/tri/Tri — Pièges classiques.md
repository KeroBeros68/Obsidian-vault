#algo #tri #pièges #erreurs #debugging

## 🪤 Piège 1 — Quick sort avec pivot fixe sur tableau trié

```python
# ❌ Pivot = dernier élément sur tableau trié → O(n²)
arr = list(range(1000))     # déjà trié
quicksort(arr, 0, 999)      # 1000 niveaux de récursion, RecursionError en Python

# ✅ Pivot aléatoire : élimine le worst case avec haute probabilité
import random
def partition_random(arr, low, high):
    idx = random.randint(low, high)
    arr[idx], arr[high] = arr[high], arr[idx]
    return partition_lomuto(arr, low, high)
```

> [!warning] Entrées adversariales
> En compétition ou en production, une entrée triée ou inversée est courante. Sans pivot aléatoire ou médiane de trois, quick sort est O(n²) et peut stack overflow.

---

## 🪤 Piège 2 — Merge sort instable par un mauvais opérateur de comparaison

```python
# ❌ < strict au lieu de <= lors de la fusion → instabilité
def merge(left, right):
    ...
    if left[i] < right[j]:     # ❌ si égaux, on prend right avant left
        result.append(left[i])
    ...

# ✅ <= pour garantir la stabilité (priorité à gauche si égaux)
    if left[i] <= right[j]:    # ✅ left passe en premier si égalité
        result.append(left[i])
```

> [!tip] Règle mémo
> Dans la fusion, l'opérateur `<=` donne la priorité à l'élément de gauche (qui vient d'un sous-tableau antérieur) → stabilité garantie.

---

## 🪤 Piège 3 — Counting sort avec valeurs non bornées ou négatives

```python
# ❌ Ignorer les valeurs négatives
arr   = [3, -1, 4, -2, 5]
count = [0] * (max(arr) + 1)    # taille 6, mais indices -1, -2 → IndexError ❌

# ✅ Décaler par min_val
min_val = min(arr)               # -2
count   = [0] * (max(arr) - min_val + 1)
for x in arr:
    count[x - min_val] += 1     # ✅ index toujours positif
```

> [!warning] k implicitement grand
> `arr = [0, 1_000_000]` → k = 1 000 001. Counting sort alloue un tableau de 1M entrées pour 2 éléments. Vérifier que k est raisonnable avant d'utiliser counting sort.

---

## 🪤 Piège 4 — Radix sort instable à cause d'un counting sort instable

```python
# ❌ Placement en ordre direct (non inversé) → instabilité
for x in arr:                   # ← ordre direct
    digit         = (x // exp) % base
    count[digit] -= 1
    output[count[digit]] = x    # les éléments égaux sont renversés ❌

# ✅ Parcourir en reversed pour préserver l'ordre relatif
for x in reversed(arr):         # ← ordre inverse ✅
    digit         = (x // exp) % base
    count[digit] -= 1
    output[count[digit]] = x
```

> [!warning] Bug silencieux
> Sans `reversed`, radix sort produit souvent un résultat trié mais avec un ordre relatif incorrect sur les égaux. Le bug ne se voit que si on trie des objets composites (ex : trier des personnes par âge puis par nom).

---

## 🪤 Piège 5 — Confondre stabilité et résultat trié

```python
# Le résultat peut être "trié" sans être stable
data = [('Bob', 2), ('Alice', 2), ('Charlie', 1)]

# Après tri instable par valeur :
# [('Charlie', 1), ('Alice', 2), ('Bob', 2)]  ← valeurs triées ✅
#                   ↑               ↑
# Mais Alice et Bob ont échangé leur ordre ! ❌ (Bob était avant Alice)

# Problème réel : tri par âge, puis par nom
# Si le tri par nom est instable, le tri précédent par âge est perdu.
```

> [!tip] Mémo
> Stabilité = les égaux gardent leur **ordre d'apparition dans l'entrée**. Crucial pour les tris multi-critères en cascade.

---

## 🪤 Piège 6 — Heap sort non stable utilisé là où la stabilité est requise

```python
# ❌ Heap sort pour trier des objets où l'ordre d'origine compte
students = [('Alice', 'A'), ('Bob', 'A'), ('Charlie', 'B')]
heap_sort(students)   # Alice et Bob peuvent être inversés ❌

# ✅ Utiliser merge sort (ou sorted() Python qui est Timsort, stable)
students.sort(key=lambda x: x[1])  # ✅ stable en Python
```

> [!info] Python sorted() est toujours stable
> `sorted()` et `.sort()` utilisent Timsort, garanti stable. En Java, `Arrays.sort()` sur objets utilise aussi merge sort (stable), mais sur primitifs utilise dual-pivot quicksort (non stable).

---

## 🪤 Piège 7 — Oublier que la construction du tas est O(n), pas O(n log n)

```python
# ❌ Pensée incorrecte : "n insertions dans un tas = O(n log n)"
for i in range(n):
    heapq.heappush(heap, arr[i])   # ⚠️ O(n log n)

# ✅ Build heap direct depuis un tableau = O(n)
heap = arr[:]
heapq.heapify(heap)                # ✅ O(n) — via sift-down depuis n//2 vers 0
```

> [!tip] Pourquoi heapify est O(n)
> Les nœuds proches des feuilles font peu de travail. La somme Σ h·(n/2^h) converge vers O(n). Voir [[Tri 04 — Tri par tas (Heap Sort)]] pour la démonstration.

---

---

## 🪤 Piège 8 — Bubble sort sans flag sur tableau trié

```python
# ❌ Sans flag : n² comparaisons même sur tableau trié
def bubble_sort_naive(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n - i - 1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
# Sur [1,2,3,4,5] : effectue quand même 10 comparaisons inutiles ❌

# ✅ Avec flag : détecte l'absence de swap → break → O(n)
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(n - i - 1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        if not swapped:
            break                    # ✅ O(n) sur tableau trié
```

> [!tip] Mémo
> Toujours ajouter le flag `swapped`. C'est 2 lignes qui transforment le best case de O(n²) en O(n).

---

## 🪤 Piège 9 — Confondre "peu d'échanges" et "rapide"

```python
# Selection sort : seulement n-1 échanges ✅
# Mais toujours n²/2 comparaisons ❌

# Sur un tableau de 10 000 éléments :
# Selection sort : ~50 000 000 comparaisons, ~10 000 échanges
# Insertion sort : ~25 000 000 comparaisons en moyenne, ~25 000 000 décalages
# Quick sort     : ~130 000 comparaisons en moyenne

# Selection sort est utile uniquement quand les échanges sont TRÈS coûteux
# (écriture sur disque, opérations réseau) mais les lectures sont gratuites.
```

> [!warning] Ne pas confondre opérations
> Selection sort minimise les **échanges** (O(n)) mais pas les **comparaisons** (toujours O(n²)). Choisir en fonction de l'opération dominante du contexte.

---

## 🪤 Piège 10 — Insertion sort sur tableau inversé en production

```python
# ❌ Insertion sort sur données inversées → O(n²) décalages
data = list(range(10000, 0, -1))     # [10000, 9999, ..., 1]
insertion_sort(data)                  # ~50 000 000 opérations

# ✅ Vérifier la distribution avant de choisir
# Si données potentiellement inversées ou aléatoires → merge/quick sort
# Si données quasi-triées (quelques inversions) → insertion sort ✅
```

> [!tip] Mémo inversions
> Insertion sort effectue exactement autant de décalages qu'il y a d'**inversions** dans le tableau. Tableau inversé = n(n-1)/2 inversions = O(n²) décalages.

---

## 🪤 Piège 11 — Shell sort avec la séquence naïve n/2

```python
# ❌ Séquence de Shell originale : worst case O(n²)
def shell_sort_naive(arr):
    gap = len(arr) // 2
    while gap > 0:
        # ... insertion sort avec ce gap
        gap //= 2                     # ❌ gaps pairs → interfèrent mal

# ✅ Séquence de Ciura : meilleurs résultats empiriques
CIURA_GAPS = [701, 301, 132, 57, 23, 10, 4, 1]

def shell_sort(arr):
    for gap in CIURA_GAPS:
        if gap >= len(arr):
            continue
        # ... insertion sort avec ce gap ✅
```

> [!warning] Les gaps pairs sont problématiques
> Avec la séquence n/2 sur un tableau de taille 2^k, les éléments pairs et impairs ne se "voient" jamais avant la dernière passe → dégénère en O(n²) sur certaines entrées.

---

## 🪤 Piège 12 — Bucket sort avec distribution non uniforme

```python
# ❌ Données groupées autour d'une valeur → un seau surchargé
data = [0.01, 0.02, 0.03] + [0.5] * 10000 + [0.99]
bucket_sort(data, n_buckets=10)
# Seau [0.4–0.5] : 10 000 éléments → O(n²) sur ce seau ❌

# ✅ Adapter le nombre de seaux ou analyser la distribution
import statistics
if statistics.stdev(data) < 0.1:        # distribution concentrée
    # Préférer merge sort ou quick sort
    data.sort()
else:
    bucket_sort(data)
```

> [!warning] Hypothèse d'uniformité
> Bucket sort est O(n) **seulement sous l'hypothèse d'uniformité**. Toujours vérifier ou garantir la distribution avant d'utiliser bucket sort.

---

## Récapitulatif rapide — mise à jour

| Piège | Solution |
|-------|----------|
| Quick sort O(n²) sur trié | Pivot aléatoire ou médiane de trois |
| Merge sort instable | Utiliser `<=` dans la fusion |
| Counting sort avec négatifs | Décaler par `min_val` |
| Radix sort instable | `reversed(arr)` dans counting sort |
| Heap sort non stable | Utiliser merge sort ou Timsort |
| Build heap O(n log n) | Utiliser `heapify` (O(n)) |
| Bubble sort sans flag | Ajouter `swapped = False` + break |
| Selection sort "rapide" car peu de swaps | Toujours O(n²) comparaisons |
| Insertion sort sur données inversées | Vérifier la distribution préalable |
| Shell sort avec gaps n/2 | Utiliser séquence de Ciura ou Knuth |
| Bucket sort sur distribution non uniforme | Analyser la distribution d'abord |
