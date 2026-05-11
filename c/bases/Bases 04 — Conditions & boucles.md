#c #bases #conditions #boucles

## if / else

```c
int x = 10;

if (x > 0) {
    printf("positif\n");
} else if (x < 0) {
    printf("négatif\n");
} else {
    printf("zéro\n");
}
```

Les accolades sont optionnelles pour un seul statement, mais toujours les écrire évite des bugs subtils.

## switch / case

```c
int code = 2;

switch (code) {
    case 1:
        printf("un\n");
        break;       // ⚠️ sans break, on "tombe" dans le case suivant
    case 2:
        printf("deux\n");
        break;
    case 3:
    case 4:
        printf("trois ou quatre\n");   // partage intentionnel
        break;
    default:
        printf("autre\n");
}
```

`switch` accepte uniquement des valeurs entières ou `char`. Pas de `float`, pas de chaînes.

> [!warning] Oubli du `break`
> Sans `break`, l'exécution continue dans le `case` suivant (fallthrough). C'est souvent un bug. Si intentionnel, commenter `/* fallthrough */`.

## while

```c
int i = 0;
while (i < 5) {
    printf("%d\n", i);
    i++;
}
```

La condition est vérifiée **avant** chaque itération. Le corps peut ne jamais s'exécuter.

## do...while

```c
int n;
do {
    scanf("%d", &n);
} while (n < 0);   // exécuté au moins une fois
```

## for

```c
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}
// i   : initialisation (exécutée une fois)
// i<10 : condition vérifiée avant chaque itération
// i++ : mise à jour exécutée après chaque itération
```

```c
// Toutes les parties sont optionnelles
int i = 0;
for (; i < 10; ) {   // équivalent à while
    i++;
}

for (;;) { ... }     // boucle infinie
```

## break & continue

```c
for (int i = 0; i < 10; i++) {
    if (i == 3) continue;   // saute l'itération courante → passe à i=4
    if (i == 7) break;      // sort de la boucle immédiatement
    printf("%d\n", i);      // affiche 0 1 2 4 5 6
}
```

`break` et `continue` n'agissent que sur la boucle **immédiatement englobante**.

## Boucles imbriquées

```c
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("(%d,%d) ", i, j);
    }
    printf("\n");
}
```

Pour sortir de plusieurs niveaux, utiliser un `goto` ou un flag — pas de `break` multi-niveau en C.

## Valeurs de vérité

```c
// En C, tout entier non nul est vrai
if (n)          // ✅ équivalent à if (n != 0)
if (ptr)        // ✅ équivalent à if (ptr != NULL)
if (!ptr)       // ✅ équivalent à if (ptr == NULL)

// Risque classique
if (x = 5)      // ❌ affectation dans la condition — toujours vrai
if (x == 5)     // ✅
```

> [!tip] Yoda conditions pour détecter l'oubli de `==`
> `if (5 == x)` — si on écrit `if (5 = x)`, le compilateur refuse (on ne peut pas affecter à un littéral). Certains préfèrent cette convention de sécurité.
