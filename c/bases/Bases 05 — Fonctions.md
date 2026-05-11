#c #bases #fonctions

## Anatomie d'une fonction

```c
// prototype (déclaration) — dans le .h ou avant l'appel
int addition(int a, int b);

// définition
int addition(int a, int b) {
    return a + b;
}

// appel
int resultat = addition(3, 4);   // 7
```

## Fonction sans valeur de retour

```c
void afficher(int n) {
    printf("valeur : %d\n", n);
    // pas de return nécessaire (return; autorisé pour sortir tôt)
}
```

## Paramètres — passage par valeur

En C, tous les arguments sont passés **par valeur** : la fonction reçoit une copie.

```c
void doubler(int n) {
    n *= 2;   // modifie la copie locale — l'original est inchangé
}

int x = 5;
doubler(x);
printf("%d\n", x);   // 5 — pas modifié ❌ pour l'appelant
```

Pour modifier une variable de l'appelant, passer son adresse (pointeur) :

```c
void doubler(int *n) {
    *n *= 2;
}

int x = 5;
doubler(&x);
printf("%d\n", x);   // 10 ✅
```

## Valeur de retour

```c
int max(int a, int b) {
    if (a > b)
        return a;
    return b;    // une seule valeur de retour possible
}
```

Pour retourner plusieurs valeurs, utiliser un pointeur en paramètre ou une struct.

## Prototype et ordre de déclaration

```c
// Sans prototype, appeler une fonction avant sa définition → erreur de compilation
int carre(int n);   // prototype — permet d'appeler carre avant sa définition

int main(void) {
    printf("%d\n", carre(4));   // ✅
    return 0;
}

int carre(int n) {
    return n * n;
}
```

## Paramètres spéciaux — tableaux

Un tableau passé en argument devient un **pointeur** vers son premier élément. La taille se perd.

```c
void afficher(int *tab, int n) {   // ou int tab[], int n
    for (int i = 0; i < n; i++)
        printf("%d ", tab[i]);
}

int t[] = {1, 2, 3, 4};
afficher(t, 4);   // passer la taille explicitement ✅
```

## Récursivité

```c
int factorielle(int n) {
    if (n <= 1) return 1;           // cas de base — indispensable
    return n * factorielle(n - 1);  // appel récursif
}

factorielle(5);   // 5 × 4 × 3 × 2 × 1 = 120
```

> [!warning] Stack overflow
> Chaque appel récursif consomme de la pile. Sans cas de base ou avec une récursion trop profonde → débordement de pile (SIGSEGV).

## Fonction `main` — variantes

```c
int main(void)                     // sans arguments
int main(int argc, char *argv[])   // avec arguments de la ligne de commande
```

```c
// argv[0] = nom du programme, argv[1..argc-1] = arguments
int main(int argc, char *argv[]) {
    for (int i = 1; i < argc; i++)
        printf("arg %d : %s\n", i, argv[i]);
    return 0;   // 0 = succès, valeur non nulle = erreur
}
```

> [!tip] Une fonction = une responsabilité
> Une fonction courte et bien nommée est plus facile à tester et à déboguer qu'une fonction longue. Si elle fait plus d'une chose, la découper.
