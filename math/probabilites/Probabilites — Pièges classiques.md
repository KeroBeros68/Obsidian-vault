#math #probabilites #pièges #erreurs

## 🪤 Piège 1 — Confondre corrélation et causalité

```
Constat : ventes de glaces ET noyades fortement corrélées en été
❌ Conclusion fausse : manger des glaces cause des noyades
✅ Explication réelle : la chaleur (variable confondante) cause les deux
```

> [!warning] Toujours chercher une variable confondante
> Face à une corrélation forte et surprenante, se demander systématiquement s'il existe une variable cachée qui explique les deux variables observées. Voir [[Probabilites 06 — Corrélation & covariance]].

---

## 🪤 Piège 2 — Mal interpréter une p-value

```
p-value = 0.04
❌ "Il y a 4% de chances que H₀ soit vraie"
✅ "Si H₀ était vraie, il y aurait 4% de chances d'observer un résultat aussi extrême"
```

> [!warning] La p-value ne dit rien sur la probabilité de H₀
> C'est l'erreur d'interprétation la plus répandue en statistiques, y compris chez des chercheurs expérimentés. Voir [[Probabilites 08 — Tests d'hypothèse]].

---

## 🪤 Piège 3 — Confondre P(A|B) et P(B|A)

```
Un test médical fiable à 95% donne un résultat positif
❌ "J'ai donc 95% de chances d'être malade"
✅ La vraie probabilité dépend du taux de prévalence de la maladie (le prior)
   — peut être bien plus faible que 95%, voir l'exemple complet
```

> [!warning] Inverser une probabilité conditionnelle change tout
> `P(Test+|Malade)` et `P(Malade|Test+)` sont deux quantités très différentes. Voir [[Probabilites 05 — Théorème de Bayes]].

---

## 🪤 Piège 4 — Diviser par n au lieu de n−1 pour un échantillon

```python
# ❌ Sous-estime la variance réelle de la population
variance = sum((x - moyenne)**2 for x in echantillon) / len(echantillon)

# ✅ Correction de Bessel, estimateur non biaisé
variance = sum((x - moyenne)**2 for x in echantillon) / (len(echantillon) - 1)
```

> [!warning] n pour une population complète, n−1 pour un échantillon
> Diviser par `n` n'est correct que si on connaît la population entière. Sur un échantillon, `n−1` corrige le biais introduit par l'estimation de la moyenne à partir des mêmes données. Voir [[Probabilites 01 — Statistiques descriptives]].

---

## 🪤 Piège 5 — Appliquer le TCL sans vérifier ses conditions

```
❌ "n ≥ 30 donc le TCL s'applique toujours"
✅ Le TCL nécessite une variance finie ET un n suffisant — une distribution
   très asymétrique peut nécessiter bien plus que n = 30
```

> [!warning] n ≥ 30 est une convention, pas une garantie
> Pour des distributions à variance infinie ou très asymétriques, la règle "n ≥ 30" est insuffisante. Voir [[Probabilites 04 — Distributions continues]].

---

## 🪤 Piège 6 — Confondre significativité statistique et significativité pratique

```
Sur un échantillon de 1 000 000 d'observations,
une différence de 0.001% peut être "statistiquement significative" (p < 0.05)
sans avoir le moindre intérêt pratique.
```

> [!tip] Toujours regarder la taille de l'effet
> Une p-value faible ne dit rien sur l'ampleur réelle de l'effet observé — toujours examiner la taille de l'effet en complément.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Corrélation prise pour causalité | Chercher une variable confondante |
| p-value interprétée comme probabilité de H₀ | p-value = P(données \| H₀), pas l'inverse |
| P(A\|B) confondue avec P(B\|A) | Toujours repasser par le théorème de Bayes complet |
| Variance divisée par n sur un échantillon | Utiliser n−1 (correction de Bessel) |
| TCL appliqué sans vérifier ses conditions | Vérifier variance finie et symétrie de la distribution d'origine |
| p-value faible confondue avec effet important | Toujours vérifier la taille de l'effet, pas seulement la p-value |
