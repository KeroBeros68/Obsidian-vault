#dev #regex #bases

## Qu'est-ce qu'une expression régulière

Une **expression régulière** (*regex*, *regexp*) est une séquence de caractères qui définit un motif de recherche pour trouver, vérifier ou manipuler des fragments de texte. Un littéral comme `abc` correspond exactement à lui-même ; la puissance des regex vient de la combinaison de symboles qui décrivent des ensembles de chaînes possibles.

> [!info] Origines
> Le concept vient des travaux du mathématicien Stephen Cole Kleene sur les langages réguliers (années 1950, théorie des automates). Ken Thompson les a intégrées dans l'éditeur QED puis `ed` sous UNIX, popularisées ensuite par `grep` (« Global Regular Expression Print »). Perl a introduit une syntaxe plus riche dans les années 1980, standardisée par POSIX.2 (1992). Aujourd'hui, PCRE (Perl Compatible Regular Expressions) est la référence de fait dans la plupart des langages.

## Quand utiliser des regex (et quand s'abstenir)

| Cas d'usage | Exemple |
|---|---|
| Recherche complexe de texte | Trouver toutes les adresses e-mail d'un document |
| Validation d'entrées utilisateur | Vérifier le format d'un mot de passe, d'un code postal |
| Extraction de données | Isoler des IP, des dates, des URLs dans un fichier log |
| Substitution / transformation | Convertir `MM/DD/YYYY` en `YYYY-MM-DD` |
| Nettoyage de données | Supprimer des caractères indésirables, anonymiser des données |

> [!warning] Quand ne PAS utiliser de regex
> Cas simple (un `if` ou une méthode de recherche native est plus lisible) ; motif extrêmement complexe (devient illisible même pour son auteur — préférer découper le problème) ; traitement volumineux ou temps réel (une regex mal optimisée devient un goulot d'étranglement). Un remplacement en masse mal conçu peut aussi produire des erreurs difficiles à détecter — toujours tester avant d'appliquer à grande échelle.

## Littéraux et métacaractères

Un **littéral** correspond à lui-même (`abc` → "abc"). Les **métacaractères** ont une signification spéciale :

| Métacaractère | Signification |
|---|---|
| `.` | N'importe quel caractère sauf un retour à la ligne |
| `^` | Début de ligne |
| `$` | Fin de ligne |
| `*` | Zéro ou plusieurs occurrences du caractère précédent |
| `+` | Une ou plusieurs occurrences du caractère précédent |
| `?` | Zéro ou une occurrence du caractère précédent |
| `\` | Échappe un métacaractère pour l'interpréter comme littéral |

Exemple : `a.b` correspond à toute chaîne de 3 caractères commençant par "a", finissant par "b", le caractère du milieu étant libre (sauf saut de ligne).

## Classes de caractères

Une classe (entre crochets `[]`) correspond à un seul caractère parmi ceux listés.

| Syntaxe | Correspond à |
|---|---|
| `[abc]` | "a", "b" ou "c" |
| `[a-z]` | N'importe quelle minuscule |
| `[A-Z]` | N'importe quelle majuscule |
| `[0-9]` | N'importe quel chiffre |
| `[^abc]` | N'importe quel caractère SAUF "a", "b", "c" (négation avec `^` en tête) |

## Classes de caractères prédéfinies

Raccourcis pour des classes courantes, plus lisibles :

| Raccourci | Équivalent | Correspond à |
|---|---|---|
| `\d` | `[0-9]` | Un chiffre |
| `\D` | `[^0-9]` | Un non-chiffre |
| `\w` | `[a-zA-Z0-9_]` | Un caractère alphanumérique ou underscore |
| `\W` | `[^a-zA-Z0-9_]` | Un caractère non alphanumérique |
| `\s` | — | Un espace blanc (espace, tabulation, saut de ligne) |
| `\S` | — | Un caractère non blanc |

Exemple : `\d{3}` correspond à exactement trois chiffres consécutifs.

## Quantificateurs

Indiquent combien de fois l'élément précédent doit apparaître.

| Quantificateur | Signification |
|---|---|
| `*` | Zéro ou plusieurs occurrences |
| `+` | Une ou plusieurs occurrences |
| `?` | Zéro ou une occurrence |
| `{n}` | Exactement n occurrences |
| `{n,}` | Au moins n occurrences |
| `{n,m}` | Entre n et m occurrences |

Exemple : `a{2,4}` correspond à "aa", "aaa" ou "aaaa".

## Groupes et captures

Les parenthèses `()` capturent une sous-partie du texte correspondant.

| Syntaxe | Effet |
|---|---|
| `(abc)` | Correspond à "abc" et capture la sous-chaîne |
| `(?:abc)` | Correspond à "abc" sans capturer (groupe non capturant) |
| `(a\|b)` | Correspond à "a" OU "b" (alternance) |

Exemple : `(abc)+` correspond à une ou plusieurs répétitions de "abc", en capturant chaque occurrence.

## Assertions — lookahead et lookbehind

Correspondances conditionnelles qui ne consomment pas de caractères dans le résultat.

| Assertion | Syntaxe | Effet |
|---|---|---|
| Lookahead positif | `(?=abc)` | Position suivie de "abc", sans l'inclure |
| Lookahead négatif | `(?!abc)` | Position NON suivie de "abc" |
| Lookbehind positif | `(?<=abc)` | Position précédée de "abc" |
| Lookbehind négatif | `(?<!abc)` | Position NON précédée de "abc" |

Exemple : `(?<=@)\w+` capture le mot qui suit immédiatement un "@", sans inclure le "@".

> [!example] Décomposer une regex d'adresse e-mail
> `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
> - `^[a-zA-Z0-9._%+-]+` : partie avant le `@` — lettres, chiffres, points, underscores, `%`, `+`, `-`
> - `@[a-zA-Z0-9.-]+` : le `@` suivi du nom de domaine
> - `\.[a-zA-Z]{2,}$` : un point suivi d'au moins deux lettres pour le suffixe (`.com`, `.org`...)

## Prérequis & suite

- [[Regex — Index des fiches]] ← retour à l'index du module
- [[Regex 02 — grep, sed et awk en Bash]] ← application de cette syntaxe en ligne de commande
- [[Regex 03 — Le module re en Python]] ← application de cette syntaxe en Python
