#math #probabilites #fondamentaux #esperance

## Probabilité : définition de base

Une probabilité quantifie la vraisemblance d'un événement, entre 0 (impossible) et 1 (certain).

```
P(événement) = nombre de cas favorables / nombre de cas possibles   (cas équiprobables)
```

```
P(obtenir un 6 au dé) = 1/6 ≈ 0.167
```

## Variable aléatoire

Une variable aléatoire `X` associe une valeur numérique à chaque issue possible d'une expérience. Elle peut être **discrète** (valeurs dénombrables : nombre de clients, résultat d'un dé) ou **continue** (valeurs sur un intervalle : une taille, un temps d'attente).

## Espérance E[X]

L'espérance est la moyenne théorique d'une variable aléatoire, pondérée par les probabilités de chaque issue — c'est l'équivalent **théorique** de la moyenne empirique vue en [[Probabilites 01 — Statistiques descriptives]].

```
Cas discret :   E[X] = Σ xᵢ · P(X = xᵢ)
```

```
Dé à 6 faces équilibré :
E[X] = 1·(1/6) + 2·(1/6) + 3·(1/6) + 4·(1/6) + 5·(1/6) + 6·(1/6)
     = 21/6 = 3.5
```

L'espérance n'est pas nécessairement une valeur que `X` peut réellement prendre (on ne fait jamais 3.5 sur un dé) — c'est la valeur moyenne attendue sur un très grand nombre de répétitions.

## Lien avec la moyenne empirique : loi des grands nombres

À mesure que le nombre d'observations augmente, la moyenne empirique d'un échantillon converge vers l'espérance théorique de la distribution dont il est issu. C'est ce résultat — la **loi des grands nombres** — qui justifie qu'on utilise des moyennes observées pour estimer des espérances théoriques inconnues.

```
n observations d'un dé lancé :
n petit (n=5)    → moyenne très variable, peut être loin de 3.5
n grand (n=10000) → moyenne très proche de 3.5
```

## Variance théorique

```
Var(X) = E[(X − E[X])²] = E[X²] − (E[X])²
```

Même logique que la variance descriptive : mesure de dispersion autour de l'espérance, mais calculée sur la distribution théorique plutôt que sur un échantillon observé.

## Propriétés utiles de l'espérance

| Propriété | Formule |
|-----------|---------|
| Linéarité | `E[aX + b] = a·E[X] + b` |
| Somme de variables (même non indépendantes) | `E[X + Y] = E[X] + E[Y]` |
| Produit de variables **indépendantes** | `E[XY] = E[X]·E[Y]` (faux en général si dépendantes) |

## Cas particuliers

> [!warning] E[XY] ≠ E[X]·E[Y] en général
> Cette égalité ne tient que si `X` et `Y` sont indépendantes. Pour des variables corrélées, il faut passer par la covariance — voir [[Probabilites 06 — Corrélation & covariance]].

> [!tip] L'espérance comme "meilleure estimation à long terme"
> Pour décider si un jeu de hasard est avantageux, calculer son espérance : positive en moyenne sur le long terme, négative signifie qu'on perd en moyenne, même si certaines parties individuelles sont gagnantes.

> [!info] Espérance infinie ou non définie
> Certaines distributions (comme la loi de Cauchy) n'ont pas d'espérance finie — un cas limite qui a des implications directes sur la validité du théorème central limite, voir [[Probabilites 04 — Distributions continues]].
