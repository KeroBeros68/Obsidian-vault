#devops #documentation #fondamentaux

## Qui maintient quoi

Une documentation sans responsable connu se dégrade mécaniquement : personne ne se sent légitime pour la corriger, personne n'est alerté quand elle devient fausse. Trois rôles distincts évitent ce flou :

| Rôle | Responsabilité |
|------|-----------------|
| **Owner** | Maintient le document à jour, arbitre en cas de désaccord sur son contenu |
| **Contributeurs** | Proposent des modifications (ajout, correction) |
| **Reviewers** | Valident les changements avant qu'ils soient intégrés |

## Cycles de review par type de document

La fréquence de relecture dépend de la criticité et de la nature du document — un document immuable n'a pas besoin d'être revu, un runbook critique si.

| Type de document | Fréquence de review |
|---------------------|------------------------|
| Runbooks critiques | Trimestrielle + systématiquement après chaque utilisation réelle |
| Service Overviews | Semestrielle + à chaque changement majeur d'architecture |
| ADRs (décisions) | Immuable — une nouvelle décision crée une nouvelle ADR, l'ancienne n'est pas réécrite |

Voir [[Documentation 07 — Reviews et fraîcheur documentaire]] pour le détail des cadences par type de document, le processus de review en 4 étapes et son automatisation.

## Le mécanisme clé : la Definition of Done

Une équipe ne maintient à jour que ce qui est **imposé par son processus**, pas ce qui repose sur la bonne volonté individuelle. Intégrer la documentation à la Definition of Done (les critères qu'une tâche doit remplir pour être considérée terminée) la rend aussi obligatoire que les tests ou la review de code.

```
Definition of Done type :
☐ Code review effectuée
☐ Tests passants
☐ Documentation mise à jour (Service Overview / Runbook concerné)
☐ CHANGELOG renseigné
```

## Le test de documentation

Un runbook qui n'a jamais été suivi par quelqu'un d'autre que son auteur n'est pas validé : son auteur comble inconsciemment les trous par sa propre connaissance implicite du système. Le vérifier consiste à le faire exécuter, tel quel, par une personne qui ne l'a pas écrit — si elle échoue à le suivre, le document doit être corrigé, pas la personne.

## Cas particuliers

> [!warning] Piège : responsabilité floue
> Un document "de l'équipe" sans owner nommé finit par n'appartenir à personne. En cas de doute, désigner un owner explicite plutôt que de compter sur une responsabilité collective diffuse.

> [!tip] Documentation hors Definition of Done = documentation qui pourrit
> Si mettre à jour la documentation n'est pas une condition de fin de tâche, elle sera systématiquement reportée puis oubliée. Voir [[Documentation 01 — Pourquoi documenter (bus factor)]] pour le coût de cet oubli.
