#sql #insert #update #delete #ddl #dml

## INSERT — ajouter des lignes

sql

```sql
-- Une seule ligne
INSERT INTO employes (nom, age, ville, salaire)
VALUES ('Frank', 28, 'Paris', 3500);

-- Plusieurs lignes en une fois
INSERT INTO employes (nom, age, ville, salaire)
VALUES
    ('Grace', 31, 'Lyon',     4200),
    ('Henri', 25, 'Bordeaux', 3100),
    ('Iris',  29, 'Paris',    3900);

-- Depuis une autre table
INSERT INTO employes_archive
SELECT * FROM employes
WHERE date_depart IS NOT NULL;
```

## UPDATE — modifier des lignes

sql

```sql
-- Modifier une valeur
UPDATE employes
SET salaire = 3500
WHERE nom = 'Alice';

-- Modifier plusieurs colonnes
UPDATE employes
SET salaire = salaire * 1.1,
    ville   = 'Lyon'
WHERE ville = 'Paris' AND age > 30;

-- Avec calcul
UPDATE employes
SET salaire = salaire + 500
WHERE salaire < 3000;
```

> [!warning] Toujours mettre un WHERE dans UPDATE ! `UPDATE employes SET salaire = 0;` → met tous les salaires à 0 ! Tester avec SELECT avant d'UPDATE :
> 
> sql
> 
> ```sql
> SELECT * FROM employes WHERE ville = 'Paris'; -- vérifier d'abord
> UPDATE employes SET salaire = ... WHERE ville = 'Paris';
> ```

## DELETE — supprimer des lignes

sql

```sql
-- Supprimer des lignes spécifiques
DELETE FROM employes
WHERE age < 18;

-- Supprimer toutes les lignes (garder la table)
DELETE FROM employes;

-- Supprimer avec sous-requête
DELETE FROM employes
WHERE dep_id NOT IN (SELECT id FROM departements);
```

> [!warning] Toujours mettre un WHERE dans DELETE ! `DELETE FROM employes;` → vide toute la table !

## TRUNCATE — vider une table rapidement

sql

```sql
TRUNCATE TABLE employes;
-- Plus rapide que DELETE sans WHERE
-- Non annulable sur certains SGBD
-- Réinitialise les auto-incréments
```

## CREATE TABLE — créer une table

sql

```sql
CREATE TABLE employes (
    id        INTEGER      PRIMARY KEY,
    nom       VARCHAR(100) NOT NULL,
    age       INTEGER      CHECK (age >= 18),
    ville     VARCHAR(50)  DEFAULT 'Paris',
    salaire   DECIMAL(10,2),
    dep_id    INTEGER      REFERENCES departements(id),
    creé_le   TIMESTAMP    DEFAULT CURRENT_TIMESTAMP
);

-- Avec auto-incrément
id SERIAL PRIMARY KEY          -- PostgreSQL
id INTEGER PRIMARY KEY AUTOINCREMENT -- SQLite
id INT AUTO_INCREMENT PRIMARY KEY    -- MySQL
```

## Contraintes courantes

|Contrainte|Description|
|---|---|
|`PRIMARY KEY`|Identifiant unique, non NULL|
|`NOT NULL`|Valeur obligatoire|
|`UNIQUE`|Valeur unique dans la colonne|
|`DEFAULT`|Valeur par défaut|
|`CHECK`|Condition à respecter|
|`REFERENCES`|Clé étrangère|

## ALTER TABLE — modifier une table

sql

```sql
ALTER TABLE employes ADD COLUMN email VARCHAR(200);
ALTER TABLE employes DROP COLUMN email;
ALTER TABLE employes RENAME COLUMN nom TO nom_complet;
ALTER TABLE employes ALTER COLUMN salaire TYPE FLOAT;
```

## DROP TABLE — supprimer une table

sql

```sql
DROP TABLE employes;
DROP TABLE IF EXISTS employes;   -- sans erreur si n'existe pas
```

## Transactions — grouper les opérations

sql

```sql
BEGIN;                           -- début de transaction
    UPDATE comptes SET solde = solde - 100 WHERE id = 1;
    UPDATE comptes SET solde = solde + 100 WHERE id = 2;
COMMIT;                          -- valider si tout OK

-- En cas d'erreur :
ROLLBACK;                        -- annuler tout
```

> [!tip] Pourquoi les transactions ? Si la 1ère UPDATE réussit mais la 2ème échoue → incohérence ! Avec BEGIN/COMMIT → soit les deux passent, soit aucune.