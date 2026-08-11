#devops #documentation #intermédiaire

## Le problème du fourre-tout

Un seul document qui mélange architecture, procédure d'urgence et historique de décisions devient illisible : on ne sait jamais où chercher, ni à quelle fréquence le relire. Séparer la documentation en familles selon leur **fonction** et leur **rythme de mise à jour** résout ce problème.

## Les 5 familles

| Famille | Fonction (question à laquelle elle répond) | Déclencheur de mise à jour |
|---------|-----------------------------------------------|-------------------------------|
| **Cartographie** | Qu'est-ce qui existe ? | À chaque changement (nouveau service, dépréciation) |
| **Architecture** | Comment ça fonctionne ? | Lors d'une refonte |
| **Procédures** | Comment agir ? | Après un incident |
| **Référentiel** | Qui contacter ? | Arrivées / départs dans l'équipe |
| **Historique** | Pourquoi ce choix a-t-il été fait ? | Jamais (immuable) |

## Illustration : arborescence type

```
documentation/
├── services/        ← Cartographie + Architecture (overviews, schémas)
├── procedures/       ← Procédures (runbooks, checklists)
├── referentiel/      ← Référentiel (équipes, conventions, contacts)
└── decisions/        ← Historique (ADRs, postmortems)
```

Chaque dossier correspond à un rythme de vie différent : `decisions/` ne se réécrit jamais (on y ajoute), `services/` change à chaque évolution du système, `procedures/` se corrige après chaque incident réel.

## Cas particuliers

> [!warning] Piège : mélanger les familles dans un même document
> Un "wiki d'équipe" unique qui contient à la fois l'architecture, les procédures d'urgence et les décisions passées oblige à tout relire pour trouver une information, et rend impossible d'appliquer des cycles de review différenciés (voir [[Documentation 02 — Gouvernance et Definition of Done]]).

> [!tip] Le classement précède l'outil
> Décider de la structure en familles avant de choisir un outil (wiki, Docs-as-Code) évite de subir les contraintes de l'outil sur l'organisation de l'information. Voir [[Documentation 05 — Outils et maturité (Docs-as-Code vs Wiki)]].
