#c #memoire #structures #liste-chaînée #vecteur

## Vecteur dynamique (tableau à taille variable)

```c
typedef struct {
    int    *data;
    size_t  len;
    size_t  cap;
} Vec;

Vec vec_new(void) {
    return (Vec){ .data = NULL, .len = 0, .cap = 0 };
}

int vec_push(Vec *v, int val) {
    if (v->len == v->cap) {
        size_t new_cap = v->cap ? v->cap * 2 : 8;
        int *tmp = realloc(v->data, new_cap * sizeof *tmp);
        if (!tmp) return -1;
        v->data = tmp;
        v->cap  = new_cap;
    }
    v->data[v->len++] = val;
    return 0;
}

void vec_free(Vec *v) {
    free(v->data);
    v->data = NULL;
    v->len = v->cap = 0;
}
```

```c
Vec v = vec_new();
vec_push(&v, 1);
vec_push(&v, 2);
vec_push(&v, 3);
// v.data = [1, 2, 3], v.len = 3, v.cap = 8 (première capacité)
vec_free(&v);
```

## Liste chaînée simple

```c
typedef struct Node {
    int          val;
    struct Node *next;
} Node;

Node *node_new(int val) {
    Node *n = malloc(sizeof *n);
    if (!n) return NULL;
    n->val  = val;
    n->next = NULL;
    return n;
}

// Insertion en tête — O(1)
Node *list_push(Node *head, int val) {
    Node *n = node_new(val);
    if (!n) return head;
    n->next = head;
    return n;
}

// Libérer toute la liste
void list_free(Node *head) {
    while (head) {
        Node *next = head->next;
        free(head);
        head = next;
    }
}
```

## Arbre binaire

```c
typedef struct Tree {
    int          val;
    struct Tree *left;
    struct Tree *right;
} Tree;

Tree *tree_new(int val) {
    Tree *t = malloc(sizeof *t);
    if (!t) return NULL;
    *t = (Tree){ val, NULL, NULL };
    return t;
}

// Libérer — post-order pour libérer les enfants avant le parent
void tree_free(Tree *t) {
    if (!t) return;
    tree_free(t->left);
    tree_free(t->right);
    free(t);
}
```

## Comparaison des structures

| Structure | Accès | Insertion tête | Insertion fin | Mémoire |
|-----------|-------|---------------|---------------|---------|
| Vecteur | O(1) | O(n) | O(1) amorti | Contiguë — cache-friendly |
| Liste chaînée | O(n) | O(1) | O(n) | Fragmentée — mauvais cache |
| Arbre BST | O(log n) | O(log n) | O(log n) | Fragmentée |

> [!tip] Préférer le vecteur par défaut
> La mémoire contiguë est cache-friendly : le CPU préfetch les lignes de cache successives. Une liste chaînée est souvent plus lente en pratique même si sa complexité algorithmique est meilleure, à cause des cache misses.

> [!warning] Ne jamais accéder à un nœud après l'avoir libéré
> ```c
> free(node);
> printf("%d\n", node->val);   // ❌ use-after-free — UB
> ```
> Toujours sauvegarder `next` avant de libérer un nœud dans une boucle.
