#bdd #mysql #architecture #intermédiaire

## Le trajet d'une requête SQL

Quand une requête arrive, elle traverse plusieurs couches à l'intérieur de `mysqld`, chacune avec un rôle précis :

| Couche | Rôle |
|--------|------|
| **Connecteur** | Authentifie le client (plugin `caching_sha2_password` par défaut depuis MySQL 8.0), maintient la session et le thread dédié |
| **Parseur SQL** | Analyse la syntaxe de la requête et construit un arbre syntaxique (AST) — une erreur de syntaxe est détectée ici |
| **Optimiseur** | Choisit le plan d'exécution le plus efficace : quel index utiliser, dans quel ordre joindre les tables, tri ou scan complet — la couche qui détermine les performances |
| **Exécuteur** | Exécute le plan choisi en appelant l'API du moteur de stockage — c'est ici que MySQL délègue à InnoDB la lecture et l'écriture réelles |
| **Moteur de stockage (InnoDB)** | Gère le stockage physique : cache mémoire, journalisation, écriture sur disque, verrouillage des lignes, transactions ACID |

Cette architecture en couches explique pourquoi MySQL supporte plusieurs moteurs de stockage : la couche SQL (connecteur → parseur → optimiseur → exécuteur) est indépendante du moteur choisi en dessous. Dans la pratique, un seul moteur mérite d'être utilisé pour des données de production — voir [[MySQL 04 — Moteurs de stockage (InnoDB vs les autres)]].

## Cas particuliers

> [!info] L'optimiseur est la couche la plus déterminante pour les performances
> Le connecteur et le parseur ont un coût relativement fixe par requête ; c'est l'optimiseur qui décide si une requête va scanner une table entière ou utiliser un index précis — une même requête SQL peut avoir des temps d'exécution radicalement différents selon le plan choisi, sans qu'aucune ligne de SQL n'ait changé.

> [!tip] La couche SQL ne "sait" rien du moteur en dessous
> C'est cette séparation qui permet de changer le moteur d'une table (`ALTER TABLE ... ENGINE=InnoDB`) sans réécrire la moindre requête applicative — le connecteur, le parseur et l'optimiseur fonctionnent à l'identique quel que soit le moteur de stockage final.

## Pour aller plus loin

Le détail des moteurs de stockage disponibles, et pourquoi InnoDB s'impose en pratique, dans [[MySQL 04 — Moteurs de stockage (InnoDB vs les autres)]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
