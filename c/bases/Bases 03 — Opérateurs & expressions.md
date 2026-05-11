#c #bases #opérateurs #expressions

## Opérateurs arithmétiques

```c
int a = 10, b = 3;

a + b    // 13
a - b    // 7
a * b    // 30
a / b    // 3  — division entière (troncature vers zéro)
a % b    // 1  — reste (modulo)
```

> [!warning] Division entière
> `7 / 2` vaut `3`, pas `3.5`. Caster au moins un opérande pour obtenir un résultat flottant : `(double)7 / 2`.

## Opérateurs de comparaison

```c
a == b    // égal         → 0 (faux) ou 1 (vrai)
a != b    // différent
a <  b    // strictement inférieur
a >  b    // strictement supérieur
a <= b    // inférieur ou égal
a >= b    // supérieur ou égal
```

En C, il n'y a pas de type booléen natif. Toute valeur non nulle est vraie.

```c
#include <stdbool.h>   // fournit bool, true, false (C99)
bool ok = true;
```

## Opérateurs logiques

```c
a && b    // ET logique  — court-circuit : b non évalué si a est faux
a || b    // OU logique  — court-circuit : b non évalué si a est vrai
!a        // NON logique
```

```c
int x = 5;
if (x > 0 && x < 10) { ... }    // ✅ vrai si 0 < x < 10
```

## Opérateurs bit à bit

```c
a & b     // ET bit à bit
a | b     // OU bit à bit
a ^ b     // XOR bit à bit
~a        // complément à un (NOT)
a << 2    // décalage à gauche de 2 bits (× 4)
a >> 1    // décalage à droite de 1 bit (÷ 2)
```

```c
// Manipuler des flags
unsigned int flags = 0b0000;
flags |= (1 << 2);    // activer le bit 2  → 0b0100
flags &= ~(1 << 2);   // désactiver bit 2  → 0b0000
flags ^= (1 << 1);    // basculer le bit 1
int b2 = (flags >> 2) & 1;  // lire le bit 2
```

## Opérateurs d'affectation

```c
x  = 5;    // affectation simple
x += 3;    // x = x + 3
x -= 2;    // x = x - 2
x *= 4;    // x = x * 4
x /= 2;    // x = x / 2
x %= 3;    // x = x % 3
x <<= 1;   // x = x << 1
x >>= 1;   // x = x >> 1
x &= 0xFF; // x = x & 0xFF
x |= 0x01; // x = x | 0x01
x ^= 0x10; // x = x ^ 0x10
```

## Incrémentation / décrémentation

```c
int x = 5;
x++;      // post-incrément : utilise x (5), puis x devient 6
++x;      // pré-incrément  : x devient 6, puis utilise 6
x--;      // post-décrément
--x;      // pré-décrément
```

```c
int a = 5;
int b = a++;   // b = 5, a = 6
int c = ++a;   // a = 7, c = 7
```

## Opérateur ternaire

```c
int max = (a > b) ? a : b;   // si a > b alors a sinon b
```

## Priorité des opérateurs (ordre décroissant)

| Priorité | Opérateurs |
|----------|-----------|
| Haute | `()` `[]` `->` `.` |
| | `!` `~` `++` `--` `(cast)` `*` `&` `sizeof` (unaires) |
| | `*` `/` `%` |
| | `+` `-` |
| | `<<` `>>` |
| | `<` `<=` `>` `>=` |
| | `==` `!=` |
| | `&` |
| | `^` |
| | `\|` |
| | `&&` |
| | `\|\|` |
| | `?:` |
| Basse | `=` `+=` `-=` ... |

> [!tip] En cas de doute, parenthéser
> `(a + b) * c` est toujours plus lisible qu'une expression qui dépend de la mémorisation des priorités.
