#devops #documentation #avancé

## Docs-as-Code vs Wiki

Deux approches pour héberger la documentation, avec un compromis différent entre rigueur et accessibilité.

| Approche | Fonctionnement | Avantages | Limites |
|----------|-----------------|-----------|---------|
| **Docs-as-Code** | Fichiers Markdown versionnés dans Git | Versionné, reviewable (pull request), historisé comme le code | Courbe d'apprentissage (Git, Markdown) pour les profils non techniques |
| **Wiki** (Confluence, Notion) | Édition WYSIWYG en ligne | Accessible immédiatement, pas de compétence technique requise | Pas de review structurée, historique des versions moins rigoureux |

Le choix dépend du public principal : une équipe déjà dans Git gagne à rester dans Git (Docs-as-Code) ; une documentation destinée à des profils non techniques (support, produit) gagne en accessibilité avec un wiki. Voir [[Documentation 06 — Docs-as-Code, workflow et outillage]] pour le détail du pipeline (piliers, PR, CI, migration).

## Outils Docs-as-Code courants

| Outil | Usage typique |
|-------|----------------|
| **MkDocs** | Documentation technique simple, mise en place rapide |
| **Starlight** (Astro) | Sites de documentation complets, personnalisables |
| **Docusaurus** | Documentation produit, versionnée par release |
| **Antora** | Documentation multi-projets/multi-dépôts |

## Trajectoire de mise en place progressive

La maturité documentaire se construit par étapes, pas d'un coup — commencer par l'urgent avant de viser l'exhaustivité.

```
Urgence (1 jour)   → 3 services critiques : Service Overview + 1 runbook chacun
Fondations (1 semaine) → Overviews complets, 3-5 runbooks, ownership défini, outil centralisé choisi
Maturité (1 mois)   → Couverture totale, reviews trimestriels effectifs, templates standardisés, métriques d'usage
```

## Cas particuliers

> [!warning] Piège : choisir l'outil avant la gouvernance
> Adopter un outil (wiki ou Docs-as-Code) avant d'avoir défini qui maintient quoi (voir [[Documentation 02 — Gouvernance et Definition of Done]]) mène à une documentation techniquement bien hébergée mais vide ou obsolète — l'outil ne résout pas le problème de responsabilité.

> [!tip] Ne pas viser l'exhaustivité en premier
> Une documentation partielle mais à jour sur les services critiques vaut mieux qu'une documentation exhaustive mais figée. La trajectoire "Urgence → Fondations → Maturité" priorise le risque avant la couverture.
