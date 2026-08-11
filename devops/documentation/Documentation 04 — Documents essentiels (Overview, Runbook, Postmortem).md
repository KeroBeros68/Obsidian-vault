#devops #documentation #intermédiaire

## Service Overview

Carte d'identité d'un service : la première chose à lire pour comprendre ce qu'il fait et à qui s'adresser en cas de problème.

- **Owner** — qui maintient ce service
- **Criticité** — impact d'une indisponibilité
- **SLA** — engagement de disponibilité/performance
- **Architecture simplifiée** — schéma haut niveau, pas le détail d'implémentation
- **Dépendances** — services amont/aval
- **Contacts** — canal d'alerte, équipe responsable

## Runbooks

Procédure d'exécution pour une situation connue (déploiement, incident type), rédigée comme une recette de cuisine : suivable sans réfléchir, sous pression.

```
1. Vérifier X via la commande exacte : `commande --option`
2. Exécuter l'action Y
3. Valider le résultat attendu (critère précis, pas "ça a l'air bon")
4. Si ça ne marche pas → section dédiée avec le prochain recours
```

> [!tip] Un bon runbook contient des commandes exactes
> Pas "redémarrer le service" mais la commande complète, copiable-collable. L'objectif est qu'une personne qui ne connaît pas le service en détail puisse l'exécuter correctement sous stress.

## Postmortems

Analyse d'un incident après résolution, factuelle et **sans recherche de coupable**.

| Section | Contenu |
|---------|---------|
| Timeline | Chronologie factuelle des événements (détection, actions, résolution) |
| Impact | Mesuré (durée, utilisateurs affectés, coût) |
| Causes racines | Méthode des 5 Whys — creuser au-delà du symptôme immédiat |
| Actions correctives | Chacune avec un owner et une échéance, pas juste une intention |

## Checklists

Listes de vérification courtes (10-15 points) pour des moments à risque récurrents : pré-déploiement, post-incident, onboarding. Leur valeur vient de leur **brièveté** — une checklist de 50 points n'est jamais suivie jusqu'au bout.

## Cas particuliers

> [!warning] Piège : postmortem qui cherche un coupable
> Un postmortem centré sur "qui a fait l'erreur" pousse les équipes à cacher les incidents plutôt qu'à les documenter honnêtement. La question utile est "qu'est-ce qui, dans le système ou le processus, a permis cette erreur ?"

> [!info] Ces documents appartiennent aux familles de [[Documentation 03 — Les 5 familles de documentation]]
> Service Overview → Cartographie/Architecture · Runbook/Checklist → Procédures · Postmortem → Historique.
