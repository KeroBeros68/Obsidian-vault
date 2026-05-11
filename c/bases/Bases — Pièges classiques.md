#c #bases #pièges #erreurs #debugging

## 🪤 Piège 1 — Variable locale non initialisée

```c
int x;
printf("%d\n", x);   // ❌ UB — valeur quelconque, pas 0

int x = 0;           // ✅ toujours initialiser
```

> [!warning] Pas d'initialisation implicite
> Seules les variables globales et `static` locales sont initialisées à 0. Les variables locales contiennent ce qui traîne en mémoire.

---

## 🪤 Piège 2 — Confusion `=` et `==`

```c
if (x = 5) { ... }    // ❌ affectation (toujours vrai si x=5≠0) — pas une comparaison
if (x == 5) { ... }   // ✅ comparaison
```

Activer `-Wall` génère un avertissement sur ce cas.

---

## 🪤 Piège 3 — Division entière inattendue

```c
int a = 7, b = 2;
double r = a / b;        // ❌ 3.0 — division entière effectuée d'abord
double r = (double)a / b; // ✅ 3.5
```

---

## 🪤 Piège 4 — `switch` sans `break`

```c
switch (n) {
    case 1:
        printf("un\n");
        // ❌ pas de break — tombe dans case 2
    case 2:
        printf("deux\n");
        break;
}
// Pour n=1 : affiche "un" puis "deux"
```

Ajouter `break` à chaque case. Si le fallthrough est intentionnel, commenter `/* fallthrough */`.

---

## 🪤 Piège 5 — `sizeof` sur un paramètre tableau

```c
void f(int arr[]) {
    printf("%zu\n", sizeof(arr));   // ❌ 8 — taille d'un pointeur, pas du tableau
}

int t[10];
printf("%zu\n", sizeof(t));        // ✅ 40 — fonctionne uniquement sur le tableau lui-même
```

Toujours passer la taille en paramètre supplémentaire.

---

## 🪤 Piège 6 — Comparer des chaînes avec `==`

```c
char *a = "hello";
char *b = "hello";

if (a == b) { ... }          // ❌ compare les adresses, pas le contenu
if (strcmp(a, b) == 0) { }   // ✅
```

---

## 🪤 Piège 7 — `scanf` sans `&`

```c
int n;
scanf("%d", n);    // ❌ UB — n est la valeur, pas une adresse
scanf("%d", &n);   // ✅
```

---

## 🪤 Piège 8 — Buffer overflow avec `strcpy` / `scanf`

```c
char buf[8];
strcpy(buf, "une chaîne bien trop longue");   // ❌ débordement de tampon
scanf("%s", buf);                              // ❌ pas de limite de taille

strncpy(buf, src, sizeof(buf) - 1);
buf[sizeof(buf) - 1] = '\0';                  // ✅ garantir la terminaison
scanf("%7s", buf);                             // ✅ borné (sizeof - 1)
```

---

## 🪤 Piège 9 — Oublier le `\0` dans un tableau de char

```c
char s[5] = {'h','e','l','l','o'};   // ❌ pas de '\0' — printf("%s") → UB
char s[6] = {'h','e','l','l','o','\0'};  // ✅
char s[]  = "hello";                     // ✅ '\0' ajouté automatiquement
```

---

## 🪤 Piège 10 — Portée de variable et bloc `if`

```c
if (condition)
    int x = 5;        // ❌ erreur de compilation en C99+ (déclaration sans bloc)
    printf("%d\n", x); // x inaccessible ici de toute façon

if (condition) {
    int x = 5;        // ✅ dans un bloc
    printf("%d\n", x);
}
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Variable non initialisée | Toujours initialiser à la déclaration |
| `=` au lieu de `==` | Activer `-Wall`, ou yoda conditions |
| Division entière | Caster avant la division |
| `switch` sans `break` | `break` systématique ou `/* fallthrough */` |
| `sizeof` sur paramètre tableau | Passer la taille en argument |
| Comparer chaînes avec `==` | Utiliser `strcmp` |
| `scanf` sans `&` | Toujours `&var` pour les types scalaires |
| Buffer overflow | `strncpy`, `snprintf`, `scanf` borné |
| Char[] sans `'\0'` | Utiliser les littéraux `"..."` ou ajouter `'\0'` explicitement |
