#math #probabilites #bayes #fondamentaux

## Probabilité conditionnelle

`P(A|B)` se lit "probabilité de A sachant B" — la probabilité que `A` se produise, étant donné qu'on sait déjà que `B` s'est produit.

```
P(A|B) = P(A et B) / P(B)        (avec P(B) > 0)
```

## Le théorème de Bayes

Le théorème de Bayes permet d'inverser le sens d'une probabilité conditionnelle — passer de `P(B|A)` (ce qu'on observe souvent facilement) à `P(A|B)` (ce qu'on veut réellement savoir).

```
P(A|B) = [P(B|A) · P(A)] / P(B)
```

| Terme | Nom | Signification |
|-------|-----|----------------|
| `P(A)` | Prior (a priori) | Probabilité de A avant toute observation |
| `P(B\|A)` | Vraisemblance | Probabilité d'observer B si A est vrai |
| `P(B)` | Évidence | Probabilité totale d'observer B (tous cas confondus) |
| `P(A\|B)` | Posterior (a posteriori) | Probabilité de A mise à jour après avoir observé B |

## Exemple classique : test médical

```
Une maladie touche 1% de la population : P(Malade) = 0.01
Le test est fiable à 95% chez les malades : P(Test+ | Malade) = 0.95
Le test donne 5% de faux positifs : P(Test+ | Sain) = 0.05

Question : si le test est positif, quelle est la probabilité d'être réellement malade ?
```

```
P(Test+) = P(Test+|Malade)·P(Malade) + P(Test+|Sain)·P(Sain)
         = 0.95 × 0.01 + 0.05 × 0.99
         = 0.0095 + 0.0495 = 0.059

P(Malade|Test+) = [P(Test+|Malade) × P(Malade)] / P(Test+)
                 = (0.95 × 0.01) / 0.059
                 ≈ 0.161  →  seulement 16% !
```

Malgré un test fiable à 95%, la probabilité réelle d'être malade après un test positif n'est que de 16% — parce que la maladie est rare (prior faible), la majorité des tests positifs proviennent en réalité des faux positifs sur la grande population des personnes saines.

## Forme étendue avec plusieurs hypothèses

```
P(Aᵢ|B) = [P(B|Aᵢ) · P(Aᵢ)] / Σⱼ P(B|Aⱼ) · P(Aⱼ)
```

Utile quand plus de deux hypothèses sont en compétition (ex. classification multi-classes en filtrage de spam : un email peut être "promotionnel", "spam", ou "légitime").

## Cas particuliers

> [!warning] Le piège classique du test fiable
> Un test "fiable à 95%" ne signifie pas "95% de chances d'être malade si positif" — la probabilité réelle dépend fortement du prior (la prévalence de la maladie dans la population). Confondre `P(B|A)` et `P(A|B)` est une erreur fréquente, parfois appelée "erreur du prosecutor's fallacy" en contexte judiciaire.

> [!tip] Mise à jour bayésienne : le posterior devient le nouveau prior
> En présence de nouvelles données successives, le posterior calculé après une première observation devient le prior pour la suivante — c'est ce mécanisme itératif qui sous-tend les filtres anti-spam, certains systèmes de recommandation, et plus largement l'apprentissage bayésien.

> [!info] Bayes et le ML
> Le théorème de Bayes est à la base des classifieurs Naive Bayes (encore utilisés en filtrage de texte) et plus largement de l'inférence bayésienne, un paradigme alternatif à l'approche fréquentiste pour estimer des paramètres de modèles.
