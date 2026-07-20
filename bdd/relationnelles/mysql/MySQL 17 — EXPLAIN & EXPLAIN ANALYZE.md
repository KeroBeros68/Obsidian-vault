#bdd #mysql #performance #intermédiaire

## EXPLAIN : le plan prévu, sans exécution

```sql
EXPLAIN SELECT * FROM logs WHERE action = 'login' AND client_id = 3;
```

```
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | logs  | ALL  | NULL          | NULL | NULL    | NULL | 90000 | Using where |
```

`type = ALL` signifie un **full table scan** : MySQL lit les 90 000 lignes pour n'en filtrer que quelques-unes — le pire cas possible.

## EXPLAIN ANALYZE : le plan avec le temps réel

Contrairement à `EXPLAIN` seul, `EXPLAIN ANALYZE` **exécute réellement** la requête et mesure le temps de chaque étape :

```sql
EXPLAIN ANALYZE SELECT * FROM logs WHERE action = 'login' AND client_id = 3\G
```

```
-> Filter: ((logs.action = 'login') AND (logs.client_id = 3))  (cost=9044 rows=900) (actual time=0.046..12.532 rows=4985 loops=1)
    -> Table scan on logs  (cost=9044 rows=90000) (actual time=0.039..7.284 rows=90000 loops=1)
```

MySQL scanne les 90 000 lignes (*Table scan*) en 7,3 ms, puis filtre pour n'en garder que 4 985 en 12,5 ms au total. L'estimateur prévoyait 900 lignes — 5× moins que la réalité observée : les statistiques sont obsolètes (voir [[MySQL 19 — Maintenance des tables]]).

## Formats alternatifs

```sql
EXPLAIN FORMAT=TREE SELECT * FROM logs WHERE action = 'login'\G   -- plus lisible, recommandé depuis 8.0
EXPLAIN FORMAT=JSON SELECT * FROM logs WHERE action = 'login'\G   -- détail complet, utile pour les outils
```

> [!info] `EXPLAIN INTO` (MySQL 8.4+)
> Stocke le plan dans une variable réutilisable, mais exige `FORMAT=JSON` et n'est **pas** compatible avec `EXPLAIN ANALYZE` :
> ```sql
> EXPLAIN FORMAT=JSON INTO @plan SELECT * FROM logs WHERE action = 'login';
> SELECT JSON_PRETTY(@plan)\G
> ```

## Corriger : ajouter un index et comparer

```sql
CREATE INDEX idx_logs_action_client ON logs (action, client_id);

EXPLAIN ANALYZE SELECT * FROM logs WHERE action = 'login' AND client_id = 3\G
-- -> Index lookup on logs using idx_logs_action_client (actual time=0.052..2.463 rows=4985 loops=1)
```

De 12,5 ms (full table scan) à 2,5 ms (index lookup) — **5× plus rapide**, sur la même donnée.

## Les types d'accès (colonne `type`), du pire au meilleur

| Type | Signification | Performance |
|------|-------------------|-------------------|
| `ALL` | Full table scan, lit toutes les lignes | Pire |
| `index` | Full index scan, parcourt tout l'index | Mauvais |
| `range` | Scan partiel d'index (`BETWEEN`, `<`, `>`, `IN`) | Acceptable |
| `ref` | Lookup par index non-unique | Bon |
| `eq_ref` | Lookup par index unique (jointure sur clé primaire) | Très bon |
| `const` | Lookup par clé primaire, résultat unique | Optimal |
| `system` | Table à une seule ligne | Optimal |

> [!tip] La règle à retenir
> Voir `ALL` sur une table de plus de 10 000 lignes avec un `WHERE` est quasi systématiquement le signe d'un index manquant.

## Cas particuliers

> [!warning] Un index ajouté sans lire le plan peut aggraver la situation
> Voir [[MySQL 13 — Observer l'activité (PROCESSLIST & connexions)]] — toujours confirmer via `EXPLAIN`/`EXPLAIN ANALYZE` qu'un index cible effectivement le bon problème avant de le créer sur une table de production volumineuse.

## Pour aller plus loin

Une fois le plan compris, la santé interne d'InnoDB (redo log, purge, adaptive hash index) est couverte dans [[MySQL 18 — Surveiller InnoDB en profondeur]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
