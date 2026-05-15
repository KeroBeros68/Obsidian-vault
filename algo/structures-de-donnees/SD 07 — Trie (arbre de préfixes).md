#algorithmique #structures-de-données #trie #préfixes #recherche

## Principe
```
Stocke des chaînes de caractères en partageant les préfixes communs.

Mots insérés : "car", "card", "care", "cat", "bat"

        (root)
       /      \
      c         b
      |         |
      a         a
     / \        |
    r   t       t*
   / \
  *  d* e*

* = fin de mot
```

Chaque nœud représente un caractère. Le chemin racine→nœud forme un préfixe.

## Interface abstraite
```
insert(word)            → void
search(word)            → bool     (mot complet)
starts_with(prefix)     → bool     (préfixe)
delete(word)            → bool
words_with_prefix(p)    → List<str>
```

## Implémentation dans les langages courants

### Python
```python
class TrieNode:
    def __init__(self) -> None:
        self.children: dict[str, TrieNode] = {}
        self.is_end  : bool                = False

class Trie:
    def __init__(self) -> None:
        self.root = TrieNode()

    def insert(self, word: str) -> None:       # O(m) m = len(word)
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:       # O(m)
        node = self.root
        for ch in word:
            if ch not in node.children: return False
            node = node.children[ch]
        return node.is_end

    def starts_with(self, prefix: str) -> bool: # O(m)
        node = self.root
        for ch in prefix:
            if ch not in node.children: return False
            node = node.children[ch]
        return True

    def words_with_prefix(self, prefix: str) -> list[str]:
        node = self.root
        for ch in prefix:
            if ch not in node.children: return []
            node = node.children[ch]
        results: list[str] = []
        self._dfs(node, prefix, results)
        return results

    def _dfs(self, node: TrieNode, path: str, results: list[str]) -> None:
        if node.is_end: results.append(path)
        for ch, child in node.children.items():
            self._dfs(child, path + ch, results)
```

### Java
```java
class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isEnd = false;
}
class Trie {
    TrieNode root = new TrieNode();
    void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            node.children.putIfAbsent(c, new TrieNode());
            node = node.children.get(c);
        }
        node.isEnd = true;
    }
    boolean search(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            if (!node.children.containsKey(c)) return false;
            node = node.children.get(c);
        }
        return node.isEnd;
    }
}
```

### C++
```cpp
struct TrieNode {
    unordered_map<char, TrieNode*> children;
    bool isEnd = false;
};
```

### JavaScript / TypeScript
```typescript
class TrieNode {
  children = new Map<string, TrieNode>();
  isEnd = false;
}
```

## Complexités
| Opération | Complexité | Note |
|---|---|---|
| insert(word) | O(m) | m = longueur du mot |
| search(word) | O(m) | |
| starts_with(prefix) | O(m) | |
| words_with_prefix(p) | O(m + k) | k = nb résultats |
| Espace | O(n × m × Σ) | Σ = taille alphabet |

## Variantes
| Variante | Usage |
|---|---|
| **Trie compressé (Patricia Trie)** | Fusionne les nœuds avec un seul enfant — moins d'espace |
| **Trie sur bits (Binary Trie)** | XOR maximum, plus long préfixe commun |
| **Suffix Tree** | Sous-chaînes, recherche de pattern |
| **Aho-Corasick** | Trie + liens de suffixe — recherche multi-patterns O(n+m) |

## Cas d'usage classiques
```
✅ Autocomplétion (search suggestions)
✅ Correcteur orthographique
✅ Routage IP (Longest Prefix Match)
✅ Comptage de mots avec préfixe commun
✅ Problèmes XOR (binary trie)
✅ Dictionnaire de mots interdits (modération)
```

> [!tip] dict Python comme Trie implicite
> `defaultdict(lambda: defaultdict(...))` peut simuler un trie de façon compacte pour les prototypes, mais sans les méthodes `search` / `starts_with` intégrées.
