#math #probabilites #correlation #covariance

## Covariance

La covariance mesure si deux variables varient ensemble dans le même sens (positivement), en sens opposé (négativement), ou de façon indépendante (proche de zéro).

```
Cov(X,Y) = E[(X − E[X])·(Y − E[Y])]

Estimateur empirique :
Cov(X,Y) = (1/(n-1)) · Σ (xᵢ − x̄)(yᵢ − ȳ)
```

Le signe de la covariance indique la direction de la relation, mais son **ampleur** n'est pas interprétable directement : elle dépend des unités de `X` et `Y` (une covariance entre des salaires en euros et un âge en années n'est pas comparable à une covariance entre des salaires en milliers d'euros et un âge en mois).

## Corrélation (coefficient de Pearson)

La corrélation normalise la covariance pour obtenir une mesure indépendante des unités, toujours comprise entre -1 et 1.

```
r = Cov(X,Y) / (σ_X · σ_Y)
```

| Valeur de r | Interprétation |
|-------------|------------------|
| `r = 1` | Corrélation linéaire positive parfaite |
| `r = -1` | Corrélation linéaire négative parfaite |
| `r = 0` | Aucune corrélation linéaire |
| `\|r\| > 0.7` | Corrélation forte (convention courante) |
| `0.3 < \|r\| < 0.7` | Corrélation modérée |
| `\|r\| < 0.3` | Corrélation faible |

Ces seuils (0.3 / 0.7) sont des conventions pratiques largement utilisées en exploration de données — voir leur application directe avec `df.corr()` dans [[Pandas 08 — Stats & Exploration]].

## Corrélation ≠ causalité

Une forte corrélation entre deux variables ne prouve **jamais** qu'une variable cause l'autre. Trois explications alternatives possibles :

```
A corrélé avec B peut signifier :
1. A cause B
2. B cause A
3. Une troisième variable C cause à la fois A et B (variable confondante)
4. Pure coïncidence statistique (surtout sur de petits échantillons)
```

```
Exemple classique : ventes de glaces ET noyades sont fortement corrélées en été
→ ni l'une ne cause l'autre : la chaleur (variable confondante) cause les deux
```

## Limite de Pearson : relations non linéaires

Le coefficient de Pearson ne capture que les relations **linéaires**. Deux variables peuvent être parfaitement liées par une relation non linéaire (ex. `Y = X²`) tout en ayant une corrélation de Pearson proche de zéro.

```
Y = X²  sur un intervalle symétrique autour de 0
→ relation déterministe parfaite, mais r ≈ 0 (la relation n'est pas linéaire)
```

Pour détecter des relations monotones non linéaires, le coefficient de corrélation de Spearman (basé sur les rangs plutôt que les valeurs brutes) est une alternative plus robuste.

## Cas particuliers

> [!warning] Toujours visualiser avant d'interpréter un r
> Un même coefficient de corrélation peut correspondre à des nuages de points très différents (quartet d'Anscombe) — toujours visualiser la relation avant de se fier uniquement au chiffre.

> [!tip] Variable confondante : le réflexe à avoir
> Face à une corrélation forte et surprenante, se demander systématiquement s'il existe une variable cachée qui pourrait expliquer les deux variables observées, avant de conclure à un lien causal.

> [!info] Matrice de corrélation et multicolinéarité
> En ML, une matrice de corrélation sert aussi à détecter la multicolinéarité entre variables explicatives (deux features très corrélées entre elles apportent une information redondante à un modèle).
