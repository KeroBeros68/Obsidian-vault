#devops #documentation #avancé

## Une documentation obsolète est pire qu'une absence de documentation

Une documentation périmée crée une **fausse confiance** : elle donne des instructions qui ne fonctionnent plus, ce qui fait perdre du temps et peut aggraver une situation déjà critique — contrairement à l'absence de documentation, qui pousse au moins à vérifier ou demander.

## Les cinq qualités d'une documentation fraîche

| Qualité | Définition | Risque si absente |
|---------|-----------|---------------------|
| **Exacte** | Les informations sont correctes | Erreurs, incidents aggravés |
| **Complète** | Aucune étape manquante | Procédure impossible à terminer |
| **Actuelle** | Reflète l'état réel du système | Confusion, perte de temps |
| **Accessible** | Facilement retrouvable | Documentation existante mais inutilisée |
| **Testée** | Suivie récemment par quelqu'un | Bugs cachés non détectés |

> [!tip] Le test critique de fraîcheur
> Faire suivre la procédure, telle quelle, par une personne qui ne la connaît pas — voir aussi le "test de documentation" dans [[Documentation 02 — Gouvernance et Definition of Done]].

## Cadences de review recommandées

| Fréquence | Type de document |
|-----------|---------------------|
| **Mensuelle** | Runbooks d'incidents critiques, procédures de sécurité, checklists de déploiement |
| **Trimestrielle** | Runbooks d'opérations courantes, Service Overviews, guides d'onboarding |
| **Annuelle ou jamais** | ADRs (vérifier si la décision reste valide), documentation légale/conformité |

> [!warning] Les postmortems ne se mettent jamais à jour
> Un postmortem est un document historique figé au moment de sa rédaction — on n'y touche plus après publication, contrairement à un runbook ou un Service Overview.

## Le processus de review en 4 étapes

```
1. Identifier   → filtrer par date de dernière review, prioriser les documents critiques
2. Vérifier     → métadonnées à jour ? owner toujours actif ? liens fonctionnels ?
3. Valider      → tester réellement les commandes, actualiser captures d'écran et versions
4. Décider      → OK (mettre à jour last_reviewed) / à corriger (ticket, deadline 48h) / à archiver
```

## Automatiser la détection d'obsolescence

Standardiser un frontmatter permet à un script CI de repérer automatiquement les documents en retard de review :

```yaml
last_updated: 2026-08-09
last_reviewed: 2026-08-09
review_frequency: monthly   # monthly | quarterly | yearly | never
owner: "@alice"
criticality: high           # high | medium | low
```

Un script (ex. `check-doc-freshness.sh`) compare `last_reviewed` au `review_frequency` et peut ouvrir automatiquement une issue de rappel (GitHub Actions) quand le seuil est dépassé — le même principe d'outillage que [[Documentation 06 — Docs-as-Code, workflow et outillage]] (Vale, markdownlint, lychee) appliqué à la fraîcheur plutôt qu'à la syntaxe.

## Review post-incident : réflexe systématique

Après chaque incident, se poser explicitement :

- Le runbook utilisé était-il à jour ?
- Manquait-il des étapes cruciales ?
- Y avait-il des erreurs dans les commandes ?
- Le Service Overview reflète-t-il encore la réalité ?

Actions immédiates : créer un ticket, l'assigner à l'owner du document, deadline 48h, review croisée obligatoire avant fusion des corrections.

## Archivage vs suppression

Supprimer une vieille documentation détruit un historique parfois utile (compréhension d'une décision passée, audit, formation). Archiver la préserve sans la laisser polluer la documentation active :

```
docs/
├── actifs/          ← documentation vivante, sujette à review
└── archives/
    ├── 2024/
    └── 2025/         ← classement par année de retrait, pas par thème
```

Chaque document archivé porte un bandeau explicite : *"Document archivé. Raison : [...]. Référence historique uniquement."*

## Métriques de suivi

| Métrique | Cible | Seuil d'alerte |
|----------|-------|-------------------|
| % de documents à jour | > 90 % | < 80 % |
| Âge moyen depuis la dernière review | < 90 jours | > 180 jours |
| Documents corrigés après incident | 100 % | < 80 % |
| Liens cassés | 0 | > 0 |
| Documents sans date de review | 0 | > 5 % |
| Documents sans owner | 0 | > 0 |

## Cas particuliers

> [!warning] Piège : review effectuée par l'auteur seul
> Une personne qui relit son propre document reproduit le même angle mort qui a produit d'éventuelles lacunes initiales (biais de confirmation). La review doit être croisée, par quelqu'un d'autre que l'auteur.

> [!warning] Piège : supprimer plutôt qu'archiver
> La suppression est irréversible et détruit l'historique ; l'archivage (avec bandeau explicite) préserve la traçabilité sans encombrer la documentation active.

> [!info] Référence externe
> Google (SRE Workbook, *Software Engineering at Google* chap. 10) formalise ce principe sous le nom de *freshness dates* : chaque document affiche sa date et son responsable de dernière review, avec rappel automatique si le seuil est dépassé.
