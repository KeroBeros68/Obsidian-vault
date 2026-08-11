#devops #documentation #avancé

## Définition et six piliers

**Docs-as-Code** consiste à traiter la documentation exactement comme du code source : mêmes pratiques, mêmes outils, même rigueur que pour le code de production.

1. **Format texte** — Markdown, MDX, AsciiDoc : lisible par des humains, versionnable par Git
2. **Versioning Git** — historique complet, branches, tags, traçabilité
3. **Pull Requests** — proposer, discuter et approuver un changement avant qu'il ne soit intégré
4. **Reviews obligatoires** — validation humaine avant publication
5. **CI/CD** — vérifications automatiques (liens cassés, orthographe, style, build)
6. **Déploiement automatisé** — un merge déclenche la publication, sans intervention manuelle

Ces piliers s'enchaînent : le format texte permet Git, qui permet branches et PR, dont la qualité est renforcée par le CI/CD.

## Le pipeline complet

```
Édition (branche dédiée) → Pull Request → CI (lint, liens, build) → Review humaine → Merge → Déploiement auto
```

- **Édition** — sur une branche dédiée, jamais directement sur `main`, pour permettre le travail parallèle sans conflit.
- **CI** — outils typiques : `markdownlint` (syntaxe), `Vale` (style/ton/orthographe configurable), `lychee` (liens cassés), build strict du site.
- **Review humaine** — principe du **reviewer naïf** : au moins une personne qui ne connaît pas le sujet en détail doit pouvoir comprendre et suivre le document.
- **Merge = publication** — aucune étape manuelle supplémentaire entre le merge et la mise en ligne.

## Wiki vs Docs-as-Code

| Aspect | Wiki (Confluence, Notion) | Docs-as-Code |
|--------|------------------------------|----------------|
| Review avant publication | Absente — publication instantanée par n'importe qui | Obligatoire via Pull Request |
| Historique | Opaque, traçabilité limitée | Complet (`git log`, `git blame`) |
| Format des données | Propriétaire, migration difficile | Texte brut, portable |
| Travail parallèle | Impossible sans écraser les modifications d'autrui | Branches indépendantes |
| Validation automatique | Aucune | CI (lint, liens, build) |
| Édition | Éditeur web uniquement | N'importe quel éditeur, hors-ligne possible |

### Illustration — le scénario catastrophe du wiki

```
Lundi    : Alice modifie un runbook
Mardi    : Bob écrase le travail d'Alice sur le même document
Mercredi : Charlie introduit une erreur en corrigeant une typo
Jeudi    : Incident en production — le runbook contient l'erreur de Charlie
Vendredi : Post-mortem impossible — historique incomplet, personne ne sait qui a changé quoi
```

Avec Docs-as-Code, chaque modification passe par une PR reviewée, l'historique Git montre précisément qui a changé quoi et quand, et `git revert` annule n'importe quel changement problématique.

## Migration depuis un wiki existant

Stratégie progressive plutôt que big-bang :

1. Les nouveaux documents partent directement en Git.
2. Migrer en priorité les documents critiques (runbooks, procédures d'incident).
3. Laisser mourir le reste (pages non consultées depuis plusieurs mois).
4. Mettre le wiki en lecture seule, ajouter un bandeau de redirection, fixer une date de fermeture ferme.

## Cas particuliers

> [!warning] Piège : trop de friction pour contribuer
> Si proposer une correction (même une typo) exige une PR complète avec review, les contributeurs abandonnent silencieusement. Signes d'alerte : "flemme de faire une PR pour une typo", CI qui bloque pour un avertissement mineur, review qui traîne plusieurs jours. Solutions : autoriser les commits directs pour les corrections mineures, distinguer avertissements et erreurs bloquantes en CI, imposer un SLA de review (24h).

> [!warning] Anti-pattern : deux sources de vérité en parallèle
> Garder un wiki actif en même temps que la documentation Git fait que plus personne ne sait où se trouve la version fiable. Une seule source doit faire autorité ; l'autre est mise en lecture seule puis fermée.

> [!warning] Anti-pattern : reviews de façade
> Approuver une PR sans l'avoir réellement lue équivaut à ne pas avoir de review du tout. Exiger un commentaire substantif et faire tourner les reviewers limite ce risque.

> [!tip] Estimer large pour une migration
> Une migration de wiki vers Docs-as-Code prend généralement bien plus longtemps que prévu (conversion automatique imparfaite, nettoyage manuel) — multiplier l'estimation initiale par un facteur de sécurité plutôt que de la prendre au pied de la lettre.
