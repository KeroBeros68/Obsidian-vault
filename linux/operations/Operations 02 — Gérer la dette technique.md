#linux #operations #sre #dette-technique

## Qu'est-ce que la dette technique

La **dette technique** désigne l'ensemble des compromis techniques qui rendent un système plus difficile à maintenir, faire évoluer ou comprendre. Ward Cunningham a inventé cette métaphore financière en 1992 pour expliquer à des dirigeants non-techniques pourquoi le code a besoin d'être retravaillé régulièrement.

| Concept financier | Équivalent technique |
|---|---|
| Emprunt | Raccourci pris pour livrer plus vite |
| Principal | Temps nécessaire pour corriger proprement |
| Intérêts | Temps perdu à chaque modification du code endetté |
| Faillite | Système devenu impossible à maintenir |

Ces compromis sont soit **délibérés** (« on sait, mais on livre quand même ») soit **involontaires** (« on ne savait pas qu'il existait mieux »).

> [!info] Ce qui n'est PAS de la dette technique
> Un bug (erreur à corriger, pas un compromis) ; du code qu'on n'aime pas (préférence subjective) ; du vieux code qui fonctionne encore (l'âge seul n'est pas un problème) ; des réunions improductives (overhead organisationnel, pas technique).

## Le quadrant de Fowler

Martin Fowler classe la dette selon deux axes : délibéré/inadvertant et prudent/imprudent.

| Quadrant | Reconnaissance | Action recommandée |
|---|---|---|
| Délibéré + Prudent — « stratégique » | « On sait, on a un plan » | Documenter, planifier le remboursement |
| Délibéré + Imprudent — « négligente » | « On sait, on s'en fiche » | Créer immédiatement un ticket de correction |
| Inadvertant + Prudent — « d'apprentissage » | « On ne savait pas, maintenant oui » | Prioriser selon l'impact — signe normal d'un développement itératif sain |
| Inadvertant + Imprudent — « invisible » | « On ne sait pas qu'on ne sait pas » | Former l'équipe, instaurer les revues de code — invisible jusqu'à ce qu'elle explose |

## Les six types de dette technique

La dette ne se limite pas au code — elle traverse plusieurs couches du système.

| Type | Signal d'alerte typique | Comment la détecter |
|---|---|---|
| Code (la plus visible) | Duplication, fonctions de 500 lignes, nommage `temp`/`data`, code mort commenté | Analyse statique (SonarQube, ESLint), revues, complexité cyclomatique |
| Design (la plus coûteuse) | Couplage fort, abstraction à 50 méthodes, « big ball of mud », dépendances circulaires | Diagrammes de dépendances, nombre de fichiers modifiés par commit |
| Documentation (la plus négligée) | README obsolète, API non documentée, commentaires périmés, absence d'ADR | Temps d'onboarding, questions répétées, écart README/réalité |
| Infrastructure (la plus risquée) | OS obsolète, dépendances avec CVE connues, config manuelle en 47 étapes | Scanners (Trivy, Snyk), audits de dépendances |
| Tests (la plus paralysante) | Couverture faible, tests fragiles/lents/couplés | Métriques de couverture, taux d'échec CI, temps de build |
| Processus (la plus insidieuse) | Déploiement manuel, pas de revue de code, workflows incohérents | Temps de déploiement, fréquence d'incidents liés au processus |

> [!example] ADR (Architecture Decision Record)
> Document court expliquant une décision architecturale, son contexte et ses conséquences — évite de refaire les mêmes erreurs faute de mémoire collective.

## Mesurer la dette technique

Ce qui ne se mesure pas ne s'améliore pas — et des chiffres convainquent le management mieux que des opinions.

- **TDR (Technical Debt Ratio)** = (coût de remédiation / coût de développement) × 100. Ex. : 45 jours-homme pour corriger sur 300 jours-homme investis → TDR = 15%.

| TDR | Interprétation | Action |
|---|---|---|
| 0-5% | Excellent | Maintenir les bonnes pratiques |
| 5-10% | Acceptable | Planifier du remboursement régulier |
| 10-20% | Préoccupant | Allouer 20-30% du temps au remboursement |
| >20% | Critique | Arrêter les nouvelles fonctionnalités |

- **Temps de cycle** : durée entre début du travail sur une tâche et sa mise en production — une dette élevée le rallonge (plus de code à comprendre, plus de bugs, plus de tests manuels, plus de conflits).
- **Code churn** : part du code modifié moins de 3 semaines après écriture (`git log --since="3 weeks ago" --name-only --pretty=format: | sort | uniq -c | sort -rn`) — un churn élevé révèle du code écrit trop vite ou des exigences mal comprises au départ.
- **Complexité cyclomatique** : nombre de chemins indépendants dans le code (≈ nombre de `if`/`while`/`for`/`case` + 1). Seuils : 1-10 maintenable, 11-20 à envisager de découper, 21-50 refactoring recommandé, >50 urgent.
- **Indicateurs qualitatifs** : vélocité en baisse, incidents récurrents sur la même zone, onboarding qui s'allonge, peur du refactoring, dépendance à un individu clé (« attends que Marie revienne »).

## Le cycle de gestion (5 étapes)

Gérer la dette est un processus continu, comme l'entretien d'un jardin — pas un projet ponctuel.

1. **Identifier** — sources humaines (revues de code, rétrospectives, questions des nouveaux à l'onboarding), automatisées (analyse statique, métriques CI/CD, scanners de sécurité), comportementales (fichiers souvent modifiés, zones évitées depuis des années, heures supplémentaires récurrentes).
2. **Mesurer** — quantifier chaque dette identifiée sur trois axes : impact (que se passe-t-il si on ne fait rien ?), effort (combien de temps pour corriger ?), risque (conséquences si on échoue ?). Une échelle simple S/M/L/XL suffit pour démarrer.
3. **Prioriser** — matrice impact × effort : effort faible + impact élevé = quick win prioritaire ; effort élevé + impact faible = ignorer ou reporter. Ajouter la fréquence de contact et la criticité métier comme critères.
4. **Rembourser** — choisir une stratégie adaptée (voir section suivante).
5. **Prévenir** — standards de code automatisés (CI bloquante), revues systématiques, couverture de tests minimale imposée, ADR pour les décisions importantes, alertes sur les métriques (TDR, temps de build).

## Stratégies de remboursement

| Stratégie | Fonctionnement | Quand l'utiliser |
|---|---|---|
| Règle des 20% | 20% du temps de chaque sprint dédié à la dette | Intégration au quotidien, sans sprint spécial à justifier ; nécessite de la discipline individuelle |
| Sprints techniques | 1 sprint sur 4 (ou 1 semaine/mois) focalisé dette | Chantiers importants : migrations, refontes majeures ; risque de mentalité « on nettoie une fois par trimestre » |
| Règle du scout | Chaque fichier touché est un peu amélioré en passant (« laissez le campement plus propre que trouvé ») | Dette diffuse, équipe disciplinée ; ne traite pas les gros chantiers |
| Équipe Debt Busters | Équipe dédiée à temps plein (ex. Etsy) qui identifie, chiffre et accompagne le refactoring | Dette critique accumulée sur plusieurs années, budget disponible |

> [!tip] Convaincre le management
> Traduire en langage business : « on a de la dette technique » devient « chaque fonctionnalité coûte 30% de plus qu'il y a un an » ; « il faut refactoriser » devient « on peut réduire le temps de livraison de 40% » ; « on a besoin de temps » devient « investir 2 semaines maintenant économise 2 mois sur l'année ».

## Pièges classiques

> [!warning] Vouloir tout rembourser d'un coup
> Le « grand projet de nettoyage » prend toujours plus de retard que prévu, épuise l'équipe et laisse la dette s'accumuler ailleurs pendant ce temps. Préférer un flux continu (20% du temps, chaque sprint, pour toujours) à un projet ponctuel.

> [!warning] Ne mesurer que le code
> SonarQube ne voit que la dette de code — la dette de processus (déploiement manuel) ou de documentation (onboarding de 3 mois) peut coûter bien plus cher. Évaluer les six types régulièrement, avec des métriques humaines.

> [!warning] Ignorer le contexte métier
> Refactoriser un module abandonné est du temps perdu ; refactoriser le module qui génère 80% du chiffre d'affaires est critique. Prioriser par impact business, pas par « propreté » technique.

> [!warning] Blâmer les développeurs précédents
> La dette résulte presque toujours de contraintes de délai, d'exigences qui ont évolué depuis, ou d'apprentissage progressif — pas d'incompétence. Documenter sans juger plutôt que chercher un coupable.

> [!warning] Attendre la « refonte totale »
> Le mythe du « on réécrit tout proprement » prend systématiquement deux fois plus de temps que prévu, oblige à maintenir l'ancien et le nouveau en parallèle, et le nouveau système accumule sa propre dette. Refactoriser progressivement, module par module, en gardant le système fonctionnel à chaque étape.

## Prérequis & suite

- [[Operations — Index des fiches]] ← retour à l'index du module
- [[Operations 01 — Réduire le travail ingrat]] ← prérequis conceptuel : le toil est une dette opérationnelle remboursée par l'automatisation, la dette technique est son pendant sur le code et l'architecture
- [[Manques]] → Patch management, Baseline & Drift, Gestion des capacités (P4, non couverts) : suites logiques de ce module Operations
