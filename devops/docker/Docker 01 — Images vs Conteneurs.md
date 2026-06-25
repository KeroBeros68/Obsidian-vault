#devops #docker #fondamentaux

## Images vs Conteneurs

Une **image** est un modèle figé (lecture seule) ; un **conteneur** est une instance en cours d'exécution de cette image, à laquelle Docker ajoute une couche modifiable.

## Couches et union filesystem

Une image Docker n'est pas un bloc monolithique : elle est composée de **couches** (*layers*) empilées par un filesystem en union (driver `overlay2` sous Linux).

- Chaque couche correspond à un ensemble de changements (ajout, modification, suppression de fichiers).
- Les couches sont **immuables** et **partagées** entre images : si dix images utilisent la même base `ubuntu`, cette couche n'est stockée qu'une seule fois sur le disque.
- Au sommet des couches d'image (toutes en lecture seule), Docker ajoute une **couche d'écriture** propre à chaque conteneur lancé.

```
image (lecture seule)          conteneur (lecture-écriture)
┌─────────────────────┐
│ Layer 3 — app code   │   →   ┌─────────────────────────┐
│ Layer 2 — dépendances │   →   │ Couche RW du conteneur   │
│ Layer 1 — OS de base  │   →   └─────────────────────────┘
└─────────────────────┘
```

## Copy-on-write : pourquoi c'est rapide

Le conteneur ne copie **rien** au démarrage — il lit directement dans les couches de l'image.

- **Lecture** d'un fichier présent dans une couche d'image → servi directement, aucune copie.
- **Écriture** dans un fichier d'une couche d'image → le fichier est d'abord copié vers la couche du conteneur (*copy-on-write*), puis modifié là.
- **Suppression** d'un fichier d'une couche inférieure → un fichier "whiteout" est créé dans la couche du conteneur pour masquer le fichier, sans toucher à la couche d'origine.

Cette stratégie explique pourquoi démarrer un conteneur prend quelques millisecondes : aucun gros transfert de données, juste l'ajout d'une fine couche vide.

## Cas particuliers

> [!warning] La couche du conteneur disparaît avec lui
> Toutes les données écrites dans la couche en lecture-écriture d'un conteneur sont **perdues** quand ce conteneur est supprimé (`docker rm`). Pour des données qui doivent survivre, voir [[Docker 03 — Volumes & persistance]].

> [!tip] Une image = un cache de build
> Le fait que les couches soient réutilisables est la base du système de cache de `docker build` : si une couche n'a pas changé depuis le dernier build, Docker la réutilise au lieu de la reconstruire. Détails dans [[Docker 02 — Dockerfile]].

> [!info] Driver de stockage
> `overlay2` est le driver recommandé sur Linux moderne (noyau ≥ 4.0). D'anciens drivers (`aufs`, `devicemapper`) existent mais sont obsolètes sur les installations actuelles.
