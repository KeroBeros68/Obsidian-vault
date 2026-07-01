#math #probabilites #ml #nlp #softmax #rag

## Softmax : transformer des scores en distribution de probabilité

Softmax convertit un vecteur de scores bruts (logits) en une distribution de probabilité valide — toutes les valeurs deviennent positives et leur somme égale 1.

```
softmax(zᵢ) = e^(zᵢ) / Σⱼ e^(zⱼ)
```

```
Logits bruts :     [2.0, 1.0, 0.1]
Après softmax :    [0.659, 0.242, 0.099]   (somme = 1.0)
```

C'est la fonction utilisée en sortie des modèles de classification (y compris les LLM, pour choisir le prochain token) : elle transforme les scores internes du modèle en quelque chose d'interprétable comme une probabilité sur chaque option possible.

## Pourquoi l'exponentielle, pas une simple normalisation

```
Normalisation simple :  zᵢ / Σⱼ zⱼ        → échoue si des scores sont négatifs
Softmax :                e^(zᵢ) / Σⱼ e^(zⱼ)  → toujours positif, amplifie les écarts
```

L'exponentielle amplifie les différences entre scores élevés et faibles — un score légèrement plus élevé que les autres se traduit par une probabilité disproportionnellement plus grande, ce qui rend la distribution de sortie plus "décisive".

## Température : ajuster la confiance d'une distribution softmax

```
softmax_T(zᵢ) = e^(zᵢ/T) / Σⱼ e^(zⱼ/T)
```

| Température | Effet |
|-------------|-------|
| `T < 1` | Distribution plus "piquée" — le modèle favorise fortement le score le plus élevé (sorties plus déterministes) |
| `T = 1` | Softmax standard |
| `T > 1` | Distribution plus "plate" — les options sont plus équiprobables (sorties plus variées/aléatoires) |

C'est le paramètre `temperature` qu'on règle dans la plupart des interfaces de génération de texte par LLM.

## Similarité cosinus : mesurer la proximité entre deux vecteurs

La similarité cosinus mesure l'angle entre deux vecteurs, indépendamment de leur longueur — c'est la métrique standard pour comparer des embeddings (représentations vectorielles de texte) en RAG (Retrieval-Augmented Generation).

```
cos_sim(A,B) = (A · B) / (‖A‖ · ‖B‖)
```

| Valeur | Interprétation |
|--------|------------------|
| `1` | Vecteurs parfaitement alignés (même direction) |
| `0` | Vecteurs orthogonaux (aucune relation) |
| `-1` | Vecteurs opposés |

```
En RAG : chaque document est converti en vecteur (embedding).
Pour trouver les documents les plus pertinents à une question,
on calcule la similarité cosinus entre le vecteur de la question
et les vecteurs de tous les documents, puis on garde les plus proches.
```

La similarité cosinus ignore la magnitude des vecteurs (contrairement à une distance euclidienne) — utile en NLP car la longueur d'un embedding ne reflète pas forcément un sens different, alors que sa direction capture le sens sémantique.

## Perplexité : évaluer un modèle de langage

La perplexité est une métrique d'évaluation des modèles de langage — l'exponentielle de l'entropie croisée moyenne.

```
PPL(W) = exp( -(1/N) · Σᵢ log P(wᵢ | w_{<i}) )
```

Elle s'interprète comme un "facteur de branchement effectif" : une perplexité de 50 signifie que le modèle est, en moyenne, aussi incertain que s'il devait choisir uniformément parmi 50 options à chaque mot.

```
Perplexité basse (ex. 10)  → le modèle est confiant, prédictions précises
Perplexité élevée (ex. 200) → le modèle est très incertain sur la suite du texte
```

## Cas particuliers

> [!warning] La perplexité n'est comparable qu'entre modèles utilisant le même tokenizer
> Une perplexité de 20 avec un tokenizer par sous-mots (BPE) n'est pas directement comparable à une perplexité de 20 avec un tokenizer au mot entier — la granularité du découpage en tokens change l'échelle de la métrique.

> [!tip] Similarité cosinus vs distance euclidienne
> Pour des embeddings normalisés (longueur 1), la similarité cosinus et la distance euclidienne donnent un classement équivalent des résultats les plus proches — mais cosinus reste préféré par convention en NLP pour son interprétation directe en termes d'angle/direction sémantique.

> [!info] Softmax et entropie
> Une distribution softmax très "plate" (température élevée) a une entropie élevée (forte incertitude) ; une distribution très "piquée" a une entropie faible (le modèle est très confiant sur une option) — lien direct avec le calcul de perplexité.
