#math #probabilites #distributions #discret

## Loi de Bernoulli : la brique de base

Une variable de Bernoulli modélise une expérience à deux issues (succès/échec) avec une probabilité de succès `p`.

```
P(X = 1) = p     (succès)
P(X = 0) = 1 − p  (échec)

E[X] = p
Var(X) = p(1 − p)
```

## Loi binomiale : répéter des Bernoulli indépendantes

La loi binomiale modélise le nombre de succès obtenus sur `n` répétitions indépendantes d'une expérience de Bernoulli de probabilité `p`.

```
P(X = k) = C(n,k) · p^k · (1-p)^(n-k)

où C(n,k) = n! / (k!(n-k)!)   (nombre de façons de choisir k succès parmi n essais)

E[X] = n·p
Var(X) = n·p·(1-p)
```

```
Exemple : 10 lancers de pièce équilibrée, probabilité d'obtenir exactement 6 piles
n = 10, p = 0.5, k = 6
P(X = 6) = C(10,6) · 0.5^6 · 0.5^4 ≈ 0.205
```

Cas d'usage typiques : taux de conversion (sur 100 visiteurs, combien achètent ?), taux de défaut (sur 50 pièces produites, combien sont défectueuses ?).

## Loi de Poisson : compter des événements rares dans un intervalle

La loi de Poisson modélise le nombre d'événements qui se produisent dans un intervalle de temps ou d'espace fixé, quand ces événements sont rares et indépendants.

```
P(X = k) = (λ^k · e^(-λ)) / k!

E[X] = λ
Var(X) = λ
```

`λ` (lambda) représente le nombre moyen d'événements attendus sur l'intervalle considéré.

```
Exemple : un serveur reçoit en moyenne 5 requêtes par seconde (λ = 5)
P(recevoir exactement 8 requêtes dans une seconde donnée) = (5^8 · e^(-5)) / 8! ≈ 0.065
```

Cas d'usage typiques : nombre d'appels reçus par un centre d'appel par heure, nombre de pannes serveur par jour, nombre de fautes de frappe par page.

## Binomiale vs Poisson : quand utiliser laquelle

| Situation | Loi appropriée |
|-----------|-----------------|
| Nombre fixe d'essais, chaque essai a 2 issues (succès/échec) | Binomiale |
| Événements comptés sur un intervalle continu (temps, espace), sans nombre d'essais fixe | Poisson |
| `n` grand et `p` petit dans une binomiale | Poisson peut approximer la binomiale |

## Cas particuliers

> [!warning] Indépendance requise
> La loi binomiale suppose que chaque essai est indépendant des autres — si ce n'est pas le cas (ex. tirage sans remise dans une population finie), c'est la loi hypergéométrique qui s'applique, pas la binomiale.

> [!tip] Approximation Poisson de la binomiale
> Quand `n` est grand (typiquement > 50) et `p` petit (typiquement < 0.05), une binomiale(n, p) peut être approximée par une Poisson(λ = n·p) — pratique pour simplifier des calculs sur de grands volumes avec un événement rare.

> [!info] Var(X) = λ, une propriété distinctive
> Une particularité de la loi de Poisson : sa variance est égale à son espérance. Si des données comptées montrent une variance très différente de leur moyenne, c'est un signal que le modèle de Poisson n'est probablement pas adapté.
