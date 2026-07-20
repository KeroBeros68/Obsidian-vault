#bdd #fondamentaux

## Gestion des transactions

Une transaction regroupe plusieurs opérations en une seule unité : soit toutes réussissent, soit aucune n'est appliquée.

```sql
BEGIN TRANSACTION;
UPDATE comptes SET solde = solde - 500 WHERE compte_id = '123';
UPDATE comptes SET solde = solde + 500 WHERE compte_id = '456';
COMMIT;
```

Si la deuxième instruction échouait, la première serait annulée (`ROLLBACK`) plutôt que laissée appliquée seule — sans cette garantie, un virement pourrait débiter un compte sans jamais créditer l'autre.

## Contrôle d'accès

Authentification (vérifier l'identité) et autorisations (définir les actions permises) protègent les données contre un accès non voulu.

```sql
CREATE USER 'nouvel_utilisateur' WITH PASSWORD 'mot_de_passe';
GRANT SELECT, INSERT ON table_utilisateurs TO 'nouvel_utilisateur';
```

`GRANT` accorde des privilèges précis (ici lecture et ajout, pas modification ni suppression) — le principe de moindre privilège s'applique aux bases de données comme aux systèmes (voir [[Docker 08 — Sécurité des conteneurs]] pour le même principe côté conteneurs).

## Sauvegarde et restauration

```bash
pg_dump mydatabase > sauvegarde.sql
pg_restore -d mydatabase sauvegarde.sql
```

Les sauvegardes peuvent être complètes, incrémentales ou différentielles — essentielles en cas de panne, corruption, ou suppression accidentelle.

## Réplication

Copie des données d'un serveur vers un ou plusieurs autres, pour la redondance et la haute disponibilité.

| Type | Fonctionnement |
|------|-------------------|
| Master-slave | Un serveur principal accepte les écritures, un ou plusieurs serveurs secondaires répliquent en lecture seule |
| Multi-master | Plusieurs serveurs acceptent des écritures, avec synchronisation entre eux |

## Optimisation des requêtes : les index

```sql
CREATE INDEX index_nom ON table_nom (colonne_nom);
```

Un index accélère la recherche sur la colonne indexée, au prix d'un espace disque et d'un coût d'écriture supplémentaires (chaque `INSERT`/`UPDATE` doit aussi mettre à jour l'index) — voir [[BDD — Généralités]] pour le concept général.

## Mise en cache

Stocker temporairement en mémoire le résultat de requêtes fréquentes réduit le besoin d'accéder au disque — un rôle que jouent nativement les bases en mémoire comme Redis (voir [[BDD — Types de bases de données]]), ou un cache applicatif placé devant une base relationnelle classique.

## Monitoring et audit

```bash
tail -f /var/log/postgresql/postgresql.log
```

Le **monitoring** surveille performances et anomalies (temps de réponse, taux d'erreur) ; l'**audit** enregistre les activités des utilisateurs et les modifications de données — deux fonctions complémentaires, l'une orientée performance, l'autre traçabilité/conformité.

## Haute disponibilité et tolérance aux pannes

Combine réplication, clustering et bascule automatique (*failover*) pour que le service reste opérationnel malgré la défaillance d'un serveur.

## Sécurité et chiffrement

Chiffrement des données au repos (sur disque) et en transit (réseau), pour protéger contre un accès non autorisé même en cas de compromission physique ou d'interception réseau.

## Journaux de transactions

Les *transaction logs* enregistrent chaque modification apportée aux données — indispensables pour la récupération après panne et pour garantir la durabilité (le "D" d'ACID, voir [[BDD — Généralités]]) : une transaction validée reste retrouvable même après un crash, grâce à ce journal.

## Cas particuliers

> [!warning] Un index n'est jamais gratuit
> Chaque index accélère les lectures sur la colonne concernée, mais ralentit chaque écriture qui doit le maintenir à jour — multiplier les index sur une table à forte volumétrie d'écriture (logs, événements) peut dégrader les performances globales plutôt que les améliorer.

> [!info] Réplication ≠ sauvegarde
> Une réplication propage aussi les erreurs et suppressions accidentelles vers les répliques presque instantanément — elle protège contre la panne d'un serveur, pas contre une erreur applicative ou humaine. Une vraie stratégie de sauvegarde (avec des points de restauration dans le temps) reste nécessaire en complément.
