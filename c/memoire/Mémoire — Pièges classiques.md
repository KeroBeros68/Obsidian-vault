#c #memoire #pièges #erreurs #debugging

## 🪤 Piège 1 — Memory leak

```c
void f(void) {
    int *p = malloc(100 * sizeof(int));
    if (condition) return;   // ❌ fuite si on retourne ici
    // ...
    free(p);
}

// ✅ libérer sur tous les chemins de sortie
void f(void) {
    int *p = malloc(100 * sizeof(int));
    if (!p) return;
    if (condition) { free(p); return; }
    // ...
    free(p);
}
```

---

## 🪤 Piège 2 — Use-after-free

```c
free(p);
printf("%d\n", p[0]);   // ❌ UB — mémoire libérée, peut contenir n'importe quoi

free(p);
p = NULL;               // ✅ accès via p → SIGSEGV détectable au lieu d'UB silencieux
```

---

## 🪤 Piège 3 — Double free

```c
free(p);
// ... code ...
free(p);   // ❌ UB — corruption du heap

// ✅ annuler le pointeur
free(p);
p = NULL;
free(p);   // ✅ free(NULL) est no-op
```

---

## 🪤 Piège 4 — realloc écrase le pointeur en cas d'échec

```c
int *arr = malloc(10 * sizeof(int));

arr = realloc(arr, 20 * sizeof(int));   // ❌ si realloc retourne NULL :
// arr = NULL → l'ancien bloc est perdu → fuite mémoire

// ✅ sauvegarder dans un temporaire
int *tmp = realloc(arr, 20 * sizeof(int));
if (!tmp) { free(arr); return NULL; }
arr = tmp;
```

---

## 🪤 Piège 5 — Lire de la mémoire malloc non initialisée

```c
int *arr = malloc(10 * sizeof(int));
printf("%d\n", arr[0]);   // ❌ UB — valeur indéterminée

int *arr = calloc(10, sizeof(int));   // ✅ initialisé à 0
// ou
memset(arr, 0, 10 * sizeof(int));     // ✅ après malloc
```

---

## 🪤 Piège 6 — free d'un pointeur non alloué dynamiquement

```c
int arr[10];
free(arr);      // ❌ UB — arr est sur la stack

char *s = "hello";
free(s);        // ❌ UB — s est dans .rodata
```

---

## 🪤 Piège 7 — Libérer le milieu d'un tableau

```c
int *arr = malloc(10 * sizeof(int));
arr += 5;    // avancer le pointeur
free(arr);   // ❌ UB — free attend l'adresse retournée par malloc

// ✅ conserver le pointeur original
int *orig = malloc(10 * sizeof(int));
int *p = orig;
p += 5;      // utiliser p pour parcourir
free(orig);  // libérer l'original
```

---

## 🪤 Piège 8 — Libérer les nœuds dans le mauvais ordre

```c
// Liste chaînée : libérer le nœud avant de lire next
while (node) {
    free(node);               // ❌ node->next inaccessible après free
    node = node->next;
}

// ✅ sauvegarder next avant free
while (node) {
    Node *next = node->next;  // sauvegarder
    free(node);
    node = next;
}
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Memory leak | `free` sur tous les chemins de sortie, `goto cleanup` |
| Use-after-free | `p = NULL` après `free` |
| Double free | `p = NULL` après `free` |
| realloc écrase ptr | Utiliser un temporaire |
| Lecture non initialisée | `calloc` ou `memset` |
| free de stack/rodata | Uniquement les adresses retournées par malloc |
| free du milieu d'un tableau | Conserver le pointeur original |
| Nœud libéré avant next | Sauvegarder `next` avant `free` |
