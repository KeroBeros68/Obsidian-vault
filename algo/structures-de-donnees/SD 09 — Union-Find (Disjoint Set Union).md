#algorithmique #structures-de-données #union-find #dsu #graphes #kruskal

## Principe
```
Gère une partition d'éléments en ensembles disjoints.
Répond en quasi-O(1) à deux questions :
  find(x)         → quel groupe contient x ? (retourne la racine)
  union(x, y)     → fusionner les groupes de x et y
  connected(x, y) → x et y sont-ils dans le même groupe ?

État initial : chaque élément est son propre groupe
  {0} {1} {2} {3} {4}

Après union(0,1), union(2,3), union(0,2) :
  {0, 1, 2, 3} {4}
```

## Algorithme — deux optimisations essentielles

**Union par rang** — limiter la hauteur à O(log n)
```
Toujours attacher l'arbre de rang inférieur sous celui de rang supérieur.
Si rangs égaux → attacher n'importe lequel, incrémenter le rang de la racine.
```

**Compression de chemin** — aplatir vers la racine
```
À chaque find(x), faire pointer tous les nœuds du chemin directement vers la racine.
Rend les appels suivants quasi-instantanés.
```

## Implémentation dans les langages courants

### Python
```python
class UnionFind:
    def __init__(self, n: int) -> None:
        self.parent     = list(range(n))
        self.rank       = [0] * n
        self.components = n

    def find(self, x: int) -> int:
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # compression
        return self.parent[x]

    def union(self, x: int, y: int) -> bool:
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return False          # déjà connectés
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1
        return True

    def connected(self, x: int, y: int) -> bool:
        return self.find(x) == self.find(y)
```

### Java
```java
class UnionFind {
    int[] parent, rank;
    int components;

    UnionFind(int n) {
        parent = new int[n]; rank = new int[n]; components = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank[rx] < rank[ry]) { int t = rx; rx = ry; ry = t; }
        parent[ry] = rx;
        if (rank[rx] == rank[ry]) rank[rx]++;
        components--;
        return true;
    }
    boolean connected(int x, int y) { return find(x) == find(y); }
}
```

### C++
```cpp
struct UnionFind {
    vector<int> parent, rank;
    int components;
    UnionFind(int n) : parent(n), rank(n,0), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        if (parent[x]!=x) parent[x]=find(parent[x]);
        return parent[x];
    }
    bool unite(int x, int y) {
        int rx=find(x), ry=find(y);
        if (rx==ry) return false;
        if (rank[rx]<rank[ry]) swap(rx,ry);
        parent[ry]=rx;
        if (rank[rx]==rank[ry]) rank[rx]++;
        components--;
        return true;
    }
};
```

### JavaScript / TypeScript
```typescript
class UnionFind {
  private parent: number[];
  private rank:   number[];
  components:     number;

  constructor(n: number) {
    this.parent     = Array.from({length: n}, (_, i) => i);
    this.rank       = new Array(n).fill(0);
    this.components = n;
  }
  find(x: number): number {
    if (this.parent[x] !== x)
      this.parent[x] = this.find(this.parent[x]);
    return this.parent[x];
  }
  union(x: number, y: number): boolean {
    let rx = this.find(x), ry = this.find(y);
    if (rx === ry) return false;
    if (this.rank[rx] < this.rank[ry]) [rx, ry] = [ry, rx];
    this.parent[ry] = rx;
    if (this.rank[rx] === this.rank[ry]) this.rank[rx]++;
    this.components--;
    return true;
  }
  connected(x: number, y: number): boolean {
    return this.find(x) === this.find(y);
  }
}
```

## Complexités
| | Sans optim | Union par rang | Rang + compression |
|---|---|---|---|
| find | O(n) | O(log n) | O(α(n)) ≈ O(1) |
| union | O(n) | O(log n) | O(α(n)) ≈ O(1) |
| Espace | O(n) | O(n) | O(n) |

α = fonction inverse d'Ackermann. α(n) ≤ 4 pour n ≤ 10^80 — **pratiquement O(1)**.

## Applications classiques

**Kruskal — MST** → [[Graphes 07 — Kruskal — Arbre couvrant minimal]]
```
Trier les arêtes par poids croissant
Pour chaque arête (u, v, poids) :
  si NOT connected(u, v) :
    union(u, v)
    ajouter au MST
  (si connected → créerait un cycle → ignorer)
```

**Détecter un cycle dans un graphe non orienté**
```
Pour chaque arête (u, v) :
  si connected(u, v) → cycle détecté !
  sinon              → union(u, v)
```

**Compter les composantes connexes**
```
Initialiser UnionFind(n)
Pour chaque arête (u, v) : union(u, v)
Résultat : uf.components
```

**Problème des îles (grille 2D)**
```
Indexer (r, c) → r * cols + c
Pour chaque cellule "1" :
  union avec ses voisins "1"
Résultat : nb de racines distinctes parmi les "1"
```

> [!important] Connexion graphes
> Union-Find est la structure centrale de **Kruskal** (MST) et de la **détection de cycles**.
> Sans Union-Find, Kruskal serait O(V²). Avec Union-Find : O(E log E).
> → [[Graphes 07 — Kruskal — Arbre couvrant minimal]]
