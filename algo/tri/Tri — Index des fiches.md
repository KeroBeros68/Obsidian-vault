#algo #tri #référence

## Fiches disponibles

### Fondamentaux

- [[Tri 01 — Analyse de complexité worst-average-best]]
- [[Tri 07 — Tri à bulles (Bubble Sort)]]
- [[Tri 08 — Tri par sélection (Selection Sort)]]
- [[Tri 09 — Tri par insertion (Insertion Sort)]]

### Intermédiaire

- [[Tri 02 — Tri fusion (Merge Sort)]]
- [[Tri 03 — Tri rapide (Quick Sort)]]
- [[Tri 04 — Tri par tas (Heap Sort)]]
- [[Tri 10 — Tri Shell (Shell Sort)]]
- [[Tri 11 — Tri cocktail (Cocktail Shaker Sort)]]

### Avancé (tris linéaires)

- [[Tri 05 — Tri par comptage (Counting Sort)]]
- [[Tri 06 — Tri par base (Radix Sort)]]
- [[Tri 12 — Tri par paquets (Bucket Sort)]]

### Hybrides (production)

- [[Tri 13 — Timsort]]

### Référence

- [[Tri — Glossaire]]
- [[Tri — Pièges classiques]]

## Carte mentale rapide

```
Besoin de trier ?
│
├── Entiers bornés (k petit)    → Counting Sort  O(n+k)
├── Entiers machine (int32/64)  → Radix Sort     O(d·n)
├── Flottants uniformes         → Bucket Sort    O(n) espérance
│
└── Tri par comparaisons
    ├── Stabilité requise       → Merge Sort     O(n log n) garanti
    ├── Mémoire contrainte O(1) → Heap Sort      O(n log n) garanti
    ├── Usage général           → Quick Sort     O(n log n) moyen
    ├── Données réelles/runs    → Timsort        O(n)–O(n log n)
    ├── Embarqué (pas récursion)→ Shell Sort     O(n^1.25)
    └── Très petit tableau      → Insertion Sort O(n) sur quasi-trié
```

## Complexités en un coup d'œil

| Algorithme | Best | Average | Worst | Espace | Stable |
|------------|------|---------|-------|--------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Shell Sort | O(n log n) | O(n^1.25) | O(n^1.5) | O(1) | ❌ |
| Cocktail Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ |
| Radix Sort | O(d·n) | O(d·n) | O(d·n) | O(n+b) | ✅ |
| Bucket Sort | O(n) | O(n+k) | O(n²) | O(n) | ✅ |
| Timsort | O(n) | O(n log n) | O(n log n) | O(n) | ✅ |

## Prérequis & suite

- [[Manques]] ← Structures de données (P2, non couvert) — prérequis : tas, files, tableaux
- [[Graphes — Index des fiches]] ← suite logique (O(n log n) de Kruskal, tri des arêtes)
