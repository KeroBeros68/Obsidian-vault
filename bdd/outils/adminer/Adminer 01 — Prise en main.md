#bdd #adminer #fondamentaux #interface

## L'écran de connexion

À l'ouverture, Adminer affiche un formulaire de connexion classique :

| Champ | Rôle |
|-------|------|
| Système | Moteur cible (MySQL, PostgreSQL, SQLite, MS SQL...) — pilote le driver utilisé |
| Serveur | Hôte:port, ou chemin du fichier pour SQLite |
| Utilisateur / Mot de passe | Identifiants du compte applicatif ou d'administration |
| Base de données | Optionnel — laisser vide pour choisir après connexion |

> [!info] Adminer ne stocke aucun mot de passe
> Les identifiants ne sont conservés que le temps de la session PHP (cookie signé), jamais écrits sur disque — à la différence d'un fichier `config.inc.php` phpMyAdmin qui peut contenir des identifiants en clair.

## Naviguer dans une base

Une fois connecté, l'interface se structure en trois niveaux :

- **Liste des bases** de l'instance (équivalent de `SHOW DATABASES`)
- **Liste des tables/vues** d'une base sélectionnée, avec taille et nombre de lignes estimé
- **Détail d'une table** : structure (colonnes, types, index, clés étrangères), données, et onglets dédiés (déclencheurs, privilèges selon le moteur)

## Explorer et modifier la structure

Depuis l'écran d'une table, Adminer permet de :

- créer/modifier/supprimer des colonnes, avec type, valeur par défaut, nullabilité
- ajouter des index (simple, unique, full-text selon le moteur)
- déclarer des clés étrangères sur les moteurs qui les supportent (InnoDB, PostgreSQL...)
- visualiser un diagramme de schéma relationnel généré automatiquement

> [!tip] Le SQL généré est toujours visible
> Chaque action via l'interface graphique affiche la requête SQL équivalente avant exécution — utile pour apprendre la syntaxe DDL du moteur cible ou copier la requête dans un script.

## Parcourir et éditer les données

L'onglet "Sélectionner des données" propose :

- filtres par colonne avec opérateurs (`=`, `LIKE`, `>`, `IN`...), combinables
- tri multi-colonnes, limite de lignes, pagination
- édition inline d'une cellule ou d'une ligne complète
- sélection multiple pour suppression ou édition groupée

## Exécuter du SQL directement

L'onglet "Requête SQL" accepte n'importe quelle instruction, y compris multi-requêtes séparées par `;` :

```sql
SELECT hostname, os, ram_gb FROM serveurs WHERE env = 'prod' ORDER BY ram_gb DESC;
```

Le résultat s'affiche en tableau, avec export possible directement depuis cette vue (voir [[Adminer 02 — Import & export de données]]).

> [!warning] Pas de confirmation avant un `DROP` ou un `DELETE` sans `WHERE`
> Contrairement à certains clients graphiques, l'onglet SQL exécute la requête telle quelle. Une requête destructrice tapée par erreur s'exécute immédiatement — vérifier deux fois avant de valider, en particulier avec un compte disposant de tous les privilèges.

## Pour aller plus loin

L'export/import de données (SQL, CSV) est détaillé dans [[Adminer 02 — Import & export de données]]. Avant d'ouvrir l'accès à d'autres utilisateurs, voir [[Adminer 03 — Sécurisation]].

Sources : [Adminer — site officiel](https://www.adminer.org/en/), [Adminer sur GitHub](https://github.com/vrana/adminer/)
