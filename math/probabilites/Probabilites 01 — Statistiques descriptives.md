#math #probabilites #fondamentaux #statistiques-descriptives

## Statistiques descriptives vs probabilités

Les statistiques descriptives résument des données **observées** (un échantillon réel). Les probabilités, elles, modélisent un comportement **théorique** avant même d'observer quoi que ce soit. Cette fiche pose les bases descriptives : elles servent de socle pour comprendre l'espérance et la variance théoriques de [[Probabilites 02 — Probabilités de base & espérance]].

## Mesures de tendance centrale

| Mesure | Définition | Sensibilité aux valeurs extrêmes |
|--------|------------|-----------------------------------|
| **Moyenne** | Somme des valeurs divisée par leur nombre | Forte — un outlier la déplace fortement |
| **Médiane** | Valeur qui sépare l'échantillon trié en deux moitiés égales | Faible — robuste aux outliers |
| **Mode** | Valeur la plus fréquente | Non affecté par les extrêmes, mais peu informatif pour des données continues |

```
Moyenne :   μ = (Σ xᵢ) / n

Médiane (n impair) :  valeur centrale du tableau trié
Médiane (n pair) :    moyenne des deux valeurs centrales
```

## Quartiles et IQR

```
Données triées :  Q1 ─────── Médiane (Q2) ─────── Q3
                  25%                              75%
                  └──────── IQR = Q3 − Q1 ────────┘
```

- **Q1** (premier quartile) : 25% des données lui sont inférieures.
- **Q3** (troisième quartile) : 75% des données lui sont inférieures.
- **IQR** (Inter-Quartile Range) : `Q3 − Q1`, mesure la dispersion du cœur des données, peu sensible aux outliers.

## Variance et écart-type

```
Variance population :   σ² = (Σ (xᵢ − μ)²) / N
Variance échantillon :  s² = (Σ (xᵢ − x̄)²) / (n − 1)

Écart-type = √variance
```

La variance mesure la dispersion des valeurs autour de la moyenne, mais dans une unité au carré (ex. m² si les données sont en mètres) — difficile à se représenter concrètement. L'écart-type, qui est la racine carrée de la variance, revient à l'unité d'origine (ex. des mètres) : il s'interprète directement comme "l'écart typique" entre une valeur et la moyenne

## Pourquoi n−1 et pas n pour un échantillon

Diviser par `n` sous-estime systématiquement la variance réelle de la population quand on travaille sur un échantillon : la moyenne de l'échantillon (`x̄`) est elle-même calculée à partir de ces mêmes données, ce qui "consomme" un degré de liberté. Diviser par `n − 1` (correction de Bessel) corrige ce biais et donne un estimateur non biaisé de la variance.

```
3 observations indépendantes, moyenne d'échantillon connue
→ seules 2 valeurs sont encore "libres" : la 3e est déterminée par les 2 premières + la moyenne
→ n − 1 = 2 degrés de liberté
```

## Cas particuliers

> [!warning] Moyenne trompeuse avec des outliers
> Sur des salaires très inégaux (ex. un PDG à 500k€ dans une équipe à 40k€ en moyenne), la moyenne donne une image déformée — la médiane reflète mieux la situation "typique".

> [!tip] Quand préférer la médiane
> Distribution asymétrique (revenus, prix immobilier, durées) → médiane. Distribution proche d'une normale et sans outlier → moyenne et médiane sont proches, la moyenne reste pertinente.

> [!info] La racine carrée introduit un biais résiduel
> La correction `n − 1` donne un estimateur non biaisé de la **variance**, mais sa racine carrée (l'écart-type) reste légèrement biaisée — un détail surtout pertinent pour de très petits échantillons.
