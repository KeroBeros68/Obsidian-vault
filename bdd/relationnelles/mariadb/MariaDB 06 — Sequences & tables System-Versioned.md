#bdd #mariadb #avancé

## Deux fonctionnalités que MySQL n'a pas

Sequences et tables System-Versioned sont deux fonctionnalités SQL standard implémentées par MariaDB (depuis les versions 10.3) et absentes de MySQL à ce jour. Elles répondent à des besoins classiques — générer des identifiants contrôlés, garder l'historique des modifications — sans recourir à des triggers ou à une table d'audit manuelle.

## CREATE SEQUENCE : une alternative à AUTO_INCREMENT

Une séquence est un objet autonome qui génère des valeurs à la demande, indépendamment d'une table :

```sql
CREATE SEQUENCE seq_commandes
  START WITH 1000
  INCREMENT BY 1
  MINVALUE 1
  MAXVALUE 999999999
  CACHE 100
  NOCYCLE;

SELECT NEXT VALUE FOR seq_commandes;  -- 1000
SELECT NEXT VALUE FOR seq_commandes;  -- 1001
```

| Option | Rôle |
|--------|------|
| `INCREMENT` | Pas entre deux valeurs (peut être négatif) |
| `MINVALUE` / `MAXVALUE` | Bornes de la séquence |
| `CACHE` | Nombre de valeurs pré-allouées en mémoire, accélère la génération |
| `CYCLE` / `NOCYCLE` | Recommencer à `MINVALUE` une fois `MAXVALUE` atteint, ou lever une erreur |

> [!tip] Un avantage sur AUTO_INCREMENT : une valeur partagée entre tables
> Une séquence n'est pas liée à une seule table — plusieurs tables peuvent tirer leurs identifiants de la même séquence. Elle contourne aussi une limite de `LAST_INSERT_ID()` : la valeur de n'importe quelle séquence utilisée reste accessible via `LASTVAL(seq_name)`, même après d'autres opérations entre-temps.

```sql
ALTER SEQUENCE seq_commandes RESTART WITH 5000;
DROP SEQUENCE seq_commandes;
```

## Tables System-Versioned : l'historique automatique

Ajouter `WITH SYSTEM VERSIONING` à une table active le suivi automatique de son historique : chaque `UPDATE` ou `DELETE` conserve l'ancienne version de la ligne plutôt que de l'écraser.

```sql
CREATE TABLE employes (
  id INT PRIMARY KEY,
  nom VARCHAR(100),
  salaire DECIMAL(10,2)
) WITH SYSTEM VERSIONING;
```

MariaDB ajoute deux colonnes internes invisibles, `ROW_START` et `ROW_END`, qui délimitent la période de validité de chaque version de ligne.

```sql
UPDATE employes SET salaire = 45000 WHERE id = 1;

-- État actuel
SELECT * FROM employes WHERE id = 1;

-- État historique à une date donnée
SELECT * FROM employes FOR SYSTEM_TIME AS OF '2026-01-01 00:00:00' WHERE id = 1;

-- Tout l'historique d'une ligne
SELECT *, ROW_START, ROW_END FROM employes FOR SYSTEM_TIME ALL WHERE id = 1;
```

> [!info] Usage type : audit sans table dédiée
> Une table System-Versioned répond nativement aux besoins d'audit (« quel était ce salaire il y a six mois ? »), de comparaison temporelle et de conformité réglementaire, sans écrire de trigger `AFTER UPDATE` ni maintenir une table d'historique parallèle.

```sql
-- Activer sur une table existante
ALTER TABLE employes ADD SYSTEM VERSIONING;
```

> [!warning] L'historique grossit la table indéfiniment par défaut
> Sans politique de purge, une table System-Versioned très mise à jour accumule un historique sans limite. `WITH SYSTEM VERSIONING PARTITION BY SYSTEM_TIME` permet de partitionner l'historique et d'en purger les anciennes partitions périodiquement.

## Pour aller plus loin

Le cycle de release et la politique LTS de MariaDB, à connaître avant de choisir une version pour la production, sont détaillés dans [[MariaDB 07 — Versions & cycle de support (LTS)]].

Sources : [CREATE SEQUENCE — MariaDB Documentation](https://mariadb.com/docs/server/reference/sql-structure/sequences/create-sequence), [System-Versioned Tables — MariaDB Documentation](https://mariadb.com/docs/server/reference/sql-structure/temporal-tables/system-versioned-tables)
