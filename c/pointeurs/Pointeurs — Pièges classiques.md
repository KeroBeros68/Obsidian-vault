#c #pointeurs #pièges #erreurs #debugging

## 🪤 Piège 1 — Pointeur non initialisé

```c
int *p;
*p = 42;   // ❌ UB — p contient une adresse aléatoire

int *p = NULL;   // ✅ initialiser à NULL
if (p) *p = 42;  // tester avant d'utiliser
```

---

## 🪤 Piège 2 — Retourner l'adresse d'une variable locale

```c
int *bad(void) {
    int x = 42;
    return &x;   // ❌ UB — x est détruit à la sortie de la fonction
}

int *good(void) {
    int *x = malloc(sizeof(int));
    *x = 42;
    return x;    // ✅ heap — vivant jusqu'au free
}
```

---

## 🪤 Piège 3 — `*p++` au lieu de `(*p)++`

```c
int arr[] = {1, 2, 3};
int *p = arr;

*p++;     // ❌ déréférence p, puis avance p → arr[0] non modifié
(*p)++;   // ✅ incrémente arr[0]
```

---

## 🪤 Piège 4 — sizeof sur un pointeur (confusion tableau)

```c
void f(int *arr) {
    printf("%zu\n", sizeof(arr));   // ❌ 8 (taille du pointeur) pas du tableau
}

int tab[10];
printf("%zu\n", sizeof(tab));       // ✅ 40 — sizeof fonctionne sur le tableau lui-même
```

---

## 🪤 Piège 5 — Comparer des pointeurs vers des objets distincts

```c
int x = 1, y = 2;
int *p = &x, *q = &y;

if (p < q) { ... }   // ❌ UB — comparaison de pointeurs vers objets distincts
// Seule la comparaison == et != est définie entre pointeurs non apparentés
```

---

## 🪤 Piège 6 — Cast invalide entre types incompatibles

```c
float f = 3.14f;
int *p = (int *)&f;
printf("%d\n", *p);   // ❌ UB — violation des règles d'aliasing strict

// Pour inspecter la représentation binaire :
unsigned char bytes[sizeof(float)];
memcpy(bytes, &f, sizeof(float));   // ✅ memcpy est l'exception légale
```

---

## 🪤 Piège 7 — Chaîne littérale dans char* mutable

```c
char *s = "hello";
s[0] = 'H';            // ❌ UB — .rodata en lecture seule → SIGSEGV probable

char s[] = "hello";    // ✅ copie sur la stack, modifiable
s[0] = 'H';            // ✅
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Pointeur non initialisé | `= NULL` à la déclaration |
| Retour de variable locale | Allouer sur heap ou utiliser static |
| `*p++` vs `(*p)++` | Parenthéser explicitement |
| sizeof sur param tableau | Passer la taille `n` en paramètre |
| Comparaison de pointeurs distincts | Comparer uniquement dans le même tableau |
| Cast d'aliasing | `memcpy` pour réinterprétation binaire |
| Chaîne littérale mutable | Déclarer avec `char s[] = "..."` |
