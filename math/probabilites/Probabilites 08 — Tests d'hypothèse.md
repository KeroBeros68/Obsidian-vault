#math #probabilites #tests-hypothese #avancé

## Logique générale d'un test d'hypothèse

Un test d'hypothèse confronte une hypothèse par défaut (hypothèse nulle) à une hypothèse alternative, à partir de données observées.

| Hypothèse | Signification |
|-----------|----------------|
| **H₀ (nulle)** | Position par défaut : aucun effet, aucune différence |
| **H₁ (alternative)** | Ce qu'on cherche à mettre en évidence : un effet, une différence existe |

```
Démarche générale :
1. Formuler H₀ et H₁
2. Calculer une statistique de test à partir des données
3. Calculer la p-value associée
4. Comparer la p-value à un seuil de signification α (souvent 0.05)
5. Rejeter H₀ si p-value ≤ α, sinon ne pas rejeter H₀
```

## Le t-test : comparer une ou deux moyennes

Le t-test sert à déterminer si une différence observée entre moyennes est statistiquement significative, particulièrement adapté aux petits échantillons où l'écart-type de la population est inconnu.

```
t-test à un échantillon (comparer une moyenne observée à une valeur théorique) :

t = (x̄ − μ₀) / (s / √n)

où x̄ = moyenne de l'échantillon
   μ₀ = valeur théorique de référence
   s  = écart-type de l'échantillon
   n  = taille de l'échantillon
```

```
Exemple : un fabricant prétend que ses ampoules durent 1000h en moyenne (μ₀ = 1000)
Échantillon de 25 ampoules testées : x̄ = 980h, s = 50h

t = (980 − 1000) / (50 / √25) = -20 / 10 = -2.0
```

Cette statistique `t` est ensuite comparée à une distribution de Student (proche de la normale, mais avec des "queues" plus épaisses pour tenir compte de l'incertitude sur de petits échantillons).

## La p-value : ce qu'elle signifie réellement

```
p-value = probabilité d'observer une statistique de test au moins aussi extrême
          que celle calculée, SI l'hypothèse nulle H₀ est vraie
```

```
p-value = 0.04  signifie :
"Si H₀ est vraie, il y a 4% de chances d'observer un résultat aussi extrême,
ou plus extrême, que celui obtenu, par pur hasard d'échantillonnage."
```

## Cas particuliers

> [!warning] La p-value n'est PAS la probabilité que H₀ soit vraie
> C'est l'erreur d'interprétation la plus répandue, y compris chez des chercheurs expérimentés. La p-value est calculée **en supposant H₀ vraie** — elle ne dit rien sur la probabilité que H₀ le soit réellement. Une p-value de 0.04 ne signifie pas "4% de chances de se tromper en rejetant H₀".

> [!tip] Significativité statistique ≠ significativité pratique
> Sur un échantillon très large, même une différence minime et sans intérêt pratique peut produire une p-value très faible (donc "statistiquement significative"). Toujours examiner la taille de l'effet en complément de la p-value, pas seulement le seuil de signification.

> [!info] Erreurs de type I et type II
> **Erreur de type I** : rejeter H₀ alors qu'elle est vraie (faux positif), contrôlée par le seuil `α`. **Erreur de type II** : ne pas rejeter H₀ alors qu'elle est fausse (faux négatif), liée à la puissance statistique du test. Voir [[Probabilites — Glossaire]].
