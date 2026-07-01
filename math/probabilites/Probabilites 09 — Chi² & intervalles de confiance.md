#math #probabilites #chi2 #intervalle-confiance #avancé

## Test du chi² (Khi-deux)

Le test du chi² compare des fréquences **observées** à des fréquences **attendues** sous une hypothèse donnée — typiquement pour tester l'indépendance entre deux variables catégorielles ou l'adéquation à une distribution théorique.

```
χ² = Σ (Oᵢ − Eᵢ)² / Eᵢ

où Oᵢ = fréquence observée pour la catégorie i
   Eᵢ = fréquence attendue (théorique) pour la catégorie i
```

```
Exemple : un dé est-il équilibré ?
Lancé 60 fois, fréquence attendue par face : 60/6 = 10

Face :        1    2    3    4    5    6
Observé :     8    12   9    11   7    13
Attendu :     10   10   10   10   10   10

χ² = (8-10)²/10 + (12-10)²/10 + (9-10)²/10 + (11-10)²/10 + (7-10)²/10 + (13-10)²/10
   = 0.4 + 0.4 + 0.1 + 0.1 + 0.9 + 0.9 = 2.8
```

Cette statistique est ensuite comparée à une distribution du chi², avec un nombre de degrés de liberté dépendant du nombre de catégories — une valeur faible suggère que les écarts observés sont compatibles avec le simple hasard.

## Tableau de contingence et test d'indépendance

Le chi² sert aussi à tester si deux variables catégorielles sont indépendantes, en comparant un tableau de contingence observé à celui qu'on attendrait sous indépendance totale.

```
                Préfère Produit A   Préfère Produit B
Hommes               45                  30
Femmes               35                  40
```

`H₀` : le genre et la préférence de produit sont indépendants. Une p-value faible suggère qu'il existe bien une association entre les deux variables — application directe avec `pd.crosstab()` déjà vu dans [[Pandas 08 — Stats & Exploration]].

## Intervalle de confiance pour une moyenne

Un intervalle de confiance donne une fourchette de valeurs plausibles pour un paramètre inconnu (ex. la moyenne de la population), avec un niveau de confiance donné.

```
IC = x̄ ± z(α/2) × (σ / √n)

où x̄    = moyenne de l'échantillon
   z(α/2) = valeur critique de la loi normale centrée réduite
   σ     = écart-type (de la population, ou de l'échantillon si inconnu)
   n     = taille de l'échantillon
```

| Niveau de confiance | z(α/2) |
|----------------------|--------|
| 90% | 1.645 |
| 95% | 1.96 |
| 99% | 2.576 |

```
Exemple : échantillon de 100 clients, satisfaction moyenne x̄ = 7.2/10, écart-type s = 1.5

IC à 95% = 7.2 ± 1.96 × (1.5 / √100)
         = 7.2 ± 1.96 × 0.15
         = 7.2 ± 0.294
         = [6.91 ; 7.51]
```

## Interprétation correcte d'un intervalle de confiance

```
"Si on répétait l'échantillonnage un grand nombre de fois et qu'on
construisait un intervalle de confiance à 95% à chaque fois,
95% de ces intervalles contiendraient la vraie moyenne de la population."
```

C'est une affirmation sur la **procédure**, pas sur l'intervalle obtenu une seule fois — un intervalle calculé une fois contient la vraie valeur ou ne la contient pas, il n'y a pas de "95% de chances" attachée à cet intervalle spécifique une fois calculé.

## Cas particuliers

> [!warning] σ connu vs inconnu change la distribution utilisée
> La formule avec `z` (loi normale) suppose l'écart-type de la **population** connu. En pratique, c'est rarement le cas — on utilise alors l'écart-type de l'échantillon avec la distribution de **Student** (t), plus prudente sur de petits échantillons, plutôt que la loi normale.

> [!tip] Plus l'échantillon est grand, plus l'intervalle est étroit
> La largeur de l'intervalle de confiance diminue avec `√n` au dénominateur — quadrupler la taille de l'échantillon ne fait que diviser par 2 la largeur de l'intervalle, pas par 4.

> [!info] Le chi² est sensible aux petits effectifs
> Le test du chi² perd en fiabilité quand certaines fréquences attendues sont trop faibles (règle empirique courante : `E < 5`) — dans ce cas, le test exact de Fisher est souvent préféré.
