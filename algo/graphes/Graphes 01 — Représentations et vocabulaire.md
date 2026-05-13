#algo #graphes #bases

## Graphe — définition

Un graphe G = (V, E) est un ensemble de **sommets** (V) reliés par des **arêtes** (E). Il peut être orienté ou non, pondéré ou non.

## Deux représentations fondamentales

### Liste d'adjacence

```python
# Graphe non orienté : chaque nœud pointe vers ses voisins
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D'],
    'C': ['A', 'D'],
    'D': ['B', 'C'],
}
```

### Matrice d'adjacence

```python
# Indices 0=A, 1=B, 2=C, 3=D
#         A  B  C  D
matrix = [[0, 1, 1, 0],   # A
          [1, 0, 0, 1],   # B
          [1, 0, 0, 1],   # C
          [0, 1, 1, 0]]   # D
```

## Comparaison des représentations

| Critère | Liste d'adjacence | Matrice |
|---------|------------------|---------|
| Espace | O(V + E) ✅ | O(V²) ❌ |
| Test d'arête (u,v) | O(deg(u)) | O(1) ✅ |
| Itérer les voisins | O(deg(u)) ✅ | O(V) |
| Graphe dense | ⚠️ pas d'avantage | ✅ |
| Graphe épars | ✅ | ❌ mémoire gaspillée |

## Types de graphes

| Type | Propriété |
|------|-----------|
| **Non orienté** | arête (u,v) = (v,u) |
| **Orienté (digraphe)** | arc u→v ≠ v→u |
| **Pondéré** | chaque arête a un poids w |
| **DAG** | orienté et sans cycle |
| **Connexe** | chemin entre toute paire de sommets |
| **Arbre** | connexe + sans cycle (E = V−1) |

## Illustration

```
Non orienté :       Orienté pondéré :
A — B               A —2→ B
|   |               ↑     ↓
C — D               C ←3— D
```

> [!tip] Choix de représentation
> Utiliser la liste d'adjacence par défaut. Passer à la matrice uniquement si le graphe est dense (E ≈ V²) et que les tests d'arête fréquents sont critiques.

> [!info] Complexité des parcours
> BFS et DFS sont tous deux O(V + E) avec une liste d'adjacence — on visite chaque sommet une fois et chaque arête une fois.
