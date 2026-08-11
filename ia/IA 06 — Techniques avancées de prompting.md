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

> [!info] Origine de la technique
> Le Chain-of-Thought a été formalisé par Wei et al. (2022) dans *"Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"* — l'article montre qu'ajouter quelques exemples de raisonnement explicite améliore nettement la précision sur des problèmes arithmétiques et logiques, sans toucher au modèle lui-même.

## Self-Consistency — voter sur plusieurs réponses

Générer **plusieurs fois** la même question avec du Chain-of-Thought (à température non nulle, pour obtenir des réponses variées), puis garder la réponse la plus fréquente parmi toutes celles obtenues.

```
Principe :
  Poser la même question 5 fois (température > 0, ex. 0.7)
      ↓
  Chaque réponse suit un raisonnement Chain-of-Thought
      ↓
  Compter quelle réponse finale revient le plus souvent
      ↓
  Retenir cette réponse majoritaire
```

> [!tip] Pourquoi ça marche
> Un raisonnement correct a tendance à converger vers la même réponse par plusieurs chemins différents, alors que les erreurs, elles, se dispersent dans des directions variées. Si 4 réponses sur 5 tombent sur le même résultat, c'est un signal de confiance fort ; une distribution éclatée (2/1/1/1) signale au contraire une question sur laquelle le modèle n'est pas fiable.

- ✅ Améliore nettement la fiabilité sur les problèmes où une seule réponse CoT peut se tromper
- ❌ Coûte N fois plus cher (N appels au lieu d'un seul) — à réserver aux cas où l'erreur a un coût réel

## Zero-shot, one-shot, few-shot — combien d'exemples fournir

Le nombre d'exemples donnés à l'IA forme un spectrum, pas un choix binaire :

| Technique | Principe | Idéal pour | Risque |
|---|---|---|---|
| **Zero-shot** | Aucun exemple, juste l'instruction | Tâches simples, réponses génériques | Ambiguïté sur ce que tu attends |
| **One-shot** | Un seul exemple pour cadrer le format | Un format ou un ton précis à reproduire | Un seul exemple peut ne pas suffire à fixer le pattern |
| **Few-shot** | 2 à 5 exemples | Structure spécifique, logique plus complexe | Prompt plus long (coûte plus cher en API) |

> [!tip] Commencer simple, complexifier seulement si besoin
> Toujours essayer le zero-shot en premier — c'est le moins coûteux. N'ajouter un exemple (one-shot), puis plusieurs (few-shot), que si le résultat dérive du format ou du ton attendu.

Le few-shot en pratique :

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

Ces trois patterns ont des noms reconnus dans la littérature du prompt engineering — les connaître aide à repérer la même structure d'un outil ou d'un article à l'autre.

> [!info] Persona Pattern
> C'est le rôle vu plus haut, formalisé : "Tu es un [métier] avec [X années] d'expérience." Assure cohérence et vocabulaire adapté sur toute la conversation.

> [!info] Question Refinement Pattern
> Demander à l'IA de reformuler ta question en une version plus précise *avant* d'y répondre : "À chaque question que je pose, propose d'abord une version plus précise, puis demande si tu dois l'utiliser." Utile quand tu formules mal tes propres besoins.

> [!info] Output Automater Pattern
> Demander à l'IA de générer, en plus de sa réponse, le script qui l'automatiserait : "Chaque fois que tu suggères une modification, génère aussi le script Python ou Bash pour l'appliquer." Transforme un conseil en action exécutable.

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
