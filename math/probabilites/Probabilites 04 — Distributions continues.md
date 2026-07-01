#math #probabilites #distributions #continu #tcl

## La loi normale (gaussienne)

La loi normale est la distribution continue en forme de cloche, caractérisée entièrement par sa moyenne `μ` et son écart-type `σ`.

```
Densité : f(x) = (1 / (σ√(2π))) · e^(-(x-μ)²/(2σ²))

Notation : X ~ N(μ, σ²)
```

```
        ___
       /   \
      /     \
   __/       \__
  ─────┬─────┬─────
       μ-σ   μ   μ+σ
```

## La règle 68-95-99.7

Pour une distribution normale, la proportion de données comprise dans des intervalles autour de la moyenne suit une règle fixe :

| Intervalle | Proportion des données |
|------------|--------------------------|
| `μ ± 1σ` | ≈ 68% |
| `μ ± 2σ` | ≈ 95% |
| `μ ± 3σ` | ≈ 99.7% |

Cette règle sert de base intuitive pour repérer des valeurs atypiques : une donnée à plus de 3 écarts-types de la moyenne est rare dans une distribution normale.

## Loi normale centrée réduite

Toute variable normale peut être standardisée pour se ramener à `N(0, 1)`, ce qui permet d'utiliser des tables ou fonctions statistiques universelles plutôt que de recalculer pour chaque `(μ, σ)`.

```
Z = (X − μ) / σ
```

```
Salaire moyen μ = 3000€, écart-type σ = 500€
Un salaire de 4000€ : Z = (4000 − 3000) / 500 = 2
→ ce salaire est à 2 écarts-types au-dessus de la moyenne
```

## Théorème central limite (TCL)

Le théorème central limite établit que, sous certaines conditions, la moyenne d'un grand nombre de variables aléatoires indépendantes et identiquement distribuées (i.i.d.) tend vers une loi normale — **quelle que soit la distribution d'origine** de ces variables.

```
X₁, X₂, ..., Xₙ  i.i.d., d'espérance μ et de variance σ² finie

X̄ₙ = (X₁ + X₂ + ... + Xₙ) / n

Pour n assez grand :  X̄ₙ ≈ N(μ, σ²/n)
```

C'est ce résultat qui explique pourquoi la loi normale apparaît si souvent en pratique (tailles, erreurs de mesure, moyennes d'échantillons) — et c'est le fondement théorique qui justifie les intervalles de confiance et tests d'hypothèse, voir [[Probabilites 09 — Chi² & intervalles de confiance]].

## Conditions d'application

| Condition | Détail |
|-----------|--------|
| Variance finie | Le TCL ne s'applique pas aux distributions à variance infinie (ex. loi de Cauchy) |
| Taille d'échantillon suffisante | Règle empirique courante : `n ≥ 30` pour une approximation fiable |
| Distribution d'origine symétrique | Convergence plus rapide possible dès `n = 10-15` |
| Distribution d'origine très asymétrique | Peut nécessiter un `n` plus grand que 30 pour une bonne approximation |

## Cas particuliers

> [!warning] n ≥ 30 est une règle empirique, pas une loi universelle
> Cette valeur de 30 est une convention pratique, pas un seuil mathématique strict — le bon `n` dépend de l'asymétrie de la distribution d'origine. Une distribution déjà proche de la normale converge plus vite ; une distribution très asymétrique (ex. Poisson avec petit λ) peut nécessiter davantage d'observations.

> [!tip] Si X suit déjà une loi normale
> Si chaque `Xᵢ` suit déjà une loi normale, la moyenne `X̄ₙ` est exactement normale, quel que soit `n` — pas besoin d'invoquer le TCL dans ce cas, c'est une propriété directe de la loi normale (stabilité par somme).

> [!info] TCL et machine learning
> Le TCL justifie l'utilisation de méthodes paramétriques (basées sur la normalité) même quand les données brutes ne sont pas normales, dès lors qu'on raisonne sur des moyennes ou sommes de grands échantillons — un principe sous-jacent à de nombreuses méthodes statistiques utilisées en ML.
