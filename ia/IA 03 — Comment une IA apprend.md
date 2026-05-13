#ia #bases #apprentissage #fonctionnement

## Comment une IA apprend

Le processus d'entraînement d'un modèle IA suit toujours les mêmes grandes étapes.

## Les 4 étapes de l'apprentissage

```
[1. DONNÉES] → [2. ENTRAÎNEMENT] → [3. ÉVALUATION] → [4. DÉPLOIEMENT]
```

### Étape 1 — Données

L'IA a besoin d'exemples en grande quantité.

| Type de modèle | Exemples de données d'entraînement |
|---|---|
| LLM (texte) | Des milliards de pages web, livres, articles |
| Reconnaissance d'images | Des millions d'images étiquetées |
| Traduction | Des millions de phrases en parallèle (FR ↔ EN) |

> [!warning] Garbage in, garbage out
> Si les données sont biaisées ou de mauvaise qualité, le modèle sera biaisé et peu fiable.

### Étape 2 — Entraînement

Le modèle ajuste ses **paramètres internes** (des millions de nombres) pour minimiser ses erreurs.

```
Prédiction → Comparaison avec la réalité → Calcul de l'erreur
    ↑                                               ↓
    └───────── Ajustement des paramètres ←──────────┘
                  (des milliers d'itérations)
```

> [!info] Les paramètres
> GPT-3 a 175 milliards de paramètres. Claude et GPT-4 en ont plusieurs centaines de milliards. Ce sont ces valeurs numériques qui "contiennent" le savoir du modèle.

### Étape 3 — Évaluation

On teste le modèle sur des données qu'il n'a **jamais vues** pour mesurer ses vraies performances.

- ✅ Bonne généralisation = le modèle a vraiment appris
- ❌ Surapprentissage (overfitting) = le modèle a mémorisé sans comprendre

### Étape 4 — Déploiement

Le modèle est mis à disposition. C'est à ce stade qu'on l'utilise via une interface (Claude.ai, ChatGPT) ou une API.

## Ce qui se passe quand tu envoies un message

```
Ton message (prompt)
    ↓
Tokenisation — découpage en unités (mots, syllabes)
    ↓
Traitement par le réseau de neurones
    ↓
Prédiction du token le plus probable, un par un
    ↓
Réponse affichée progressivement
```

> [!tip] Pourquoi la réponse s'affiche mot par mot ?
> Le modèle génère un token à la fois, en prédisant toujours le suivant le plus probable. C'est pour ça que la réponse "s'écrit" en direct.

> [!warning] Pas de mémoire entre sessions
> Par défaut, chaque nouvelle conversation repart de zéro. Le modèle ne "sait" pas que tu lui as parlé hier. Seul ce qui est dans la conversation en cours est accessible.
