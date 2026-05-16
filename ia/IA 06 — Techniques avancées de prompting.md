#ia #prompt #avancé #techniques

## Techniques avancées de prompting

Au-delà de la structure de base, ces techniques permettent d'obtenir des résultats bien supérieurs.

## Chain of Thought — Raisonnement étape par étape

Demande à l'IA de raisonner avant de répondre. Résultats bien meilleurs sur les problèmes complexes.

```
Réfléchis étape par étape avant de me donner ta réponse.
Explique ton raisonnement avant de conclure.
Avant de répondre, liste les éléments à considérer.
```

- ✅ Réduit les erreurs sur les problèmes logiques ou techniques
- ✅ Rend le raisonnement visible et vérifiable
- ❌ Produit des réponses plus longues

## Few-Shot Prompting — Apprendre par l'exemple

Tu donnes 2-3 exemples du résultat attendu, l'IA reproduit le pattern.

```
Transforme ces phrases en titres accrocheurs.

Exemple 1 : "Notre logiciel fait des rapports"
→ "Gagnez 2h par jour avec des rapports automatiques"

Exemple 2 : "Nous livrons vite"
→ "Livré chez vous en 24h, ou remboursé"

Maintenant transforme : "Notre application suit vos dépenses"
```

> [!tip] La technique la plus puissante
> Montrer un exemple est souvent plus efficace que l'expliquer. L'IA détecte le pattern et le reproduit fidèlement.

## Découpage des tâches complexes

Ne demande pas tout d'un coup pour un projet important. Décompose en étapes.

```
Étape 1 : "Aide-moi à définir la structure de mon business plan"
    ↓
Étape 2 : "Maintenant rédige la partie 'analyse de marché'"
    ↓
Étape 3 : "Rédige 'stratégie commerciale' en cohérence avec ce qu'on a fait"
```

- ✅ Meilleure cohérence globale
- ✅ Plus facile à corriger à chaque étape
- ✅ L'IA garde le contexte de l'étape précédente

## Verbes d'action précis

Remplace "améliore ça" par un verbe qui dit exactement quelle transformation appliquer.

| Verbe           | Ce que ça fait                                |
| --------------- | --------------------------------------------- |
| **Vulgarise**   | Rend accessible à un non-expert               |
| **Nuance**      | Ajoute des subtilités et perspectives variées |
| **Restructure** | Réorganise la logique et l'ordre des idées    |
| **Illustre**    | Enrichit avec des exemples concrets           |
| **Synthétise**  | Fusionne plusieurs idées en vision cohérente  |
| **Argumente**   | Renforce avec des preuves et de la logique    |
| **Fluidifie**   | Améliore les transitions et la cohérence      |
| **Segmente**    | Découpe en sections plus digestes             |

> [!warning] Le verbe seul ne suffit pas
> "Vulgarise" est mieux qu'"améliore" mais reste faible seul.
> "Vulgarise pour un artisan de 50 ans, sans jargon, avec des analogies du quotidien" → ✅ prompt complet.

## Prompts réutilisables

### Email professionnel
```
Tu es un expert en communication professionnelle.
Rédige un email pour [SITUATION].
Ton : [chaleureux / ferme / neutre].
Format : 3 paragraphes courts.
Termine par une question ouverte.
```

### Apprendre un sujet
```
Tu es un professeur pédagogue.
Explique-moi [SUJET] comme si j'avais 15 ans.
Utilise des analogies du quotidien.
Termine par 3 questions pour tester ma compréhension.
```

### Résoudre un problème
```
Agis comme un consultant stratégique.
Mon problème : [PROBLÈME].
Analyse la situation, identifie les causes racines,
propose 3 solutions avec avantages et inconvénients.
Sois concret.
```

> [!example] Exercice pratique
> Prends un prompt faible que tu utilises souvent et réécris-le avec : Rôle + Contexte + Tâche + Format + Contraintes. Compare les deux résultats.
