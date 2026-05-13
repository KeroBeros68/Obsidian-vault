#algo #graphes #tas-fibonacci #structures #avancé

## Tas de Fibonacci

Structure de données **paresseuse** : reporte la réorganisation au maximum pour obtenir `decrease-key` en O(1) amorti. Optimise Dijkstra et Prim sur les graphes denses.

## Propriétés

- **Forêt de tas** : collection d'arbres respectant la propriété de tas (parent ≤ enfants).
- Les arbres ne sont ni binaires ni équilibrés — aucune contrainte de forme à l'insertion.
- Après consolidation, deux arbres ne peuvent pas avoir le même degré.
- Invariant de marquage : un nœud marqué a perdu un enfant → déclenche une coupure en cascade si un second enfant est perdu.

## Complexités comparées

| Opération | Tas binaire | Tas binomial | **Tas Fibonacci** |
|-----------|-------------|--------------|-------------------|
| Insert | O(log n) | O(log n) | **O(1) amorti** |
| Find-min | O(1) | O(log n) | **O(1)** |
| Extract-min | O(log n) | O(log n) | O(log n) amorti |
| **Decrease-key** | O(log n) | O(log n) | **O(1) amorti** ← clé |
| Delete | O(log n) | O(log n) | O(log n) amorti |
| Merge | O(n) | O(log n) | **O(1)** |

## Impact sur les algorithmes de graphes

| Algorithme | Tas binaire | Tas Fibonacci |
|------------|-------------|---------------|
| Dijkstra | O((V+E) log V) | **O(E + V log V)** |
| Prim | O(E log V) | **O(E + V log V)** |

Sur un graphe dense (E ≈ V²) : O(V² log V) → **O(V²)** — gain spectaculaire.

## Les 4 opérations en détail

### Insert — O(1)

```
1. Créer un nœud isolé.
2. L'ajouter en tête de la liste circulaire des racines.
3. Mettre à jour le pointeur min si nécessaire.
→ Aucune réorganisation. Paresse totale.
```

### Extract-min — O(log n) amorti

```
1. Retirer la racine minimum.
2. Promouvoir ses enfants en racines.
3. Consolidation : fusionner les arbres de même degré
   jusqu'à ce que tous les degrés soient uniques.
4. Mettre à jour le pointeur min.
→ Consolidation coûteuse mais amortie sur toutes les insertions précédentes.
```

### Decrease-key — O(1) amorti ← raison d'être du tas Fibonacci

```
1. Réduire la valeur du nœud.
2. Si la propriété de tas est violée (nœud < parent) :
   a. Couper le nœud, le placer en racine.
   b. Marquer le parent (il a perdu un enfant).
   c. Si le parent était déjà marqué → coupure en cascade
      (remonter récursivement jusqu'à un nœud non marqué).
3. Mettre à jour le pointeur min si nécessaire.
```

### Delete — O(log n) amorti

```
1. decrease-key(nœud, −∞)   → O(1) amorti
2. extract-min()             → O(log n) amorti
```

## Analyse amortie — intuition

La fonction de **potentiel** : `Φ = nombre_arbres + 2 × noeuds_marqués`

```
Insert    : +1 arbre → ΔΦ = +1. Coût réel = O(1). Coût amorti = O(1). ✅
Extract   : consolidation réduit Φ significativement. Coût amorti = O(log n). ✅
Decrease  : +1 arbre max, −1 marqué max → ΔΦ ≤ 1. Coût réel = O(1). Coût amorti = O(1). ✅
```

L'idée : les insertions "paient à l'avance" pour la consolidation future d'extract-min.

## Illustration — forêt après insertions

```
Après 5 insertions sans extract-min :
[3] [7] [1] [9] [4]   ← 5 arbres isolés (racines)
                  ↑
              pointeur min = 1

Après extract-min(1) → consolidation :
   3               Arbres de degré différent,
  / \              aucune fusion possible ici.
 7   4
     |
     9
```

## Limitations pratiques

```
Avantages théoriques :
✅ O(1) amorti pour decrease-key
✅ Optimal pour Dijkstra/Prim sur graphes denses

Limitations en pratique :
❌ Constantes cachées importantes (beaucoup de pointeurs)
❌ Mauvaise localité cache (nœuds éparpillés en mémoire)
❌ Implémentation complexe à maintenir
⚠️ En pratique, un tas binaire suffit pour la plupart des cas
⚠️ Gain mesurable uniquement pour V,E très grands (millions de nœuds)
```

> [!tip] Analogie
> Le tas de Fibonacci est paresseux comme un étudiant qui accumule ses devoirs en tas sur son bureau et ne range tout que quand le professeur demande la note la plus basse (extract-min).

> [!warning] En production
> Préférer `heapq` (Python) ou `std::priority_queue` (C++) dans la quasi-totalité des cas. Implémenter le tas de Fibonacci uniquement si des benchmarks prouvent un goulot d'étranglement réel sur `decrease-key` à très grande échelle.

> [!info] Nom "Fibonacci"
> Les arbres générés par le tas respectent une borne inférieure de taille liée aux nombres de Fibonacci : un arbre de degré k a au moins F(k+2) nœuds. C'est ce qui garantit O(log n) pour extract-min.
