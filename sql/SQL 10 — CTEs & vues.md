#sql #cte #with #view #vues

## CTE — Common Table Expression
```sql
WITH nom_cte AS (
    SELECT ...
)
SELECT * FROM nom_cte;
```

## CTEs multiples
```sql
WITH
etape1 AS (
    SELECT * FROM employes WHERE age > 25
),
etape2 AS (
    SELECT ville, AVG(salaire) AS moy
    FROM etape1
    GROUP BY ville
)
SELECT * FROM etape2 WHERE moy > 3500;
```

> [!tip] Avantage des CTEs multiples
> Chaque étape est nommée → testable indépendamment → bien plus lisible que les sous-requêtes imbriquées

## CTE récursive — hiérarchies
```sql
WITH RECURSIVE hierarchie AS (
    -- Cas de base
    SELECT id, nom, manager_id, 1 AS niveau
    FROM employes WHERE manager_id IS NULL

    UNION ALL

    -- Cas récursif
    SELECT e.id, e.nom, e.manager_id, h.niveau + 1
    FROM employes e
    INNER JOIN hierarchie h ON e.manager_id = h.id
)
SELECT * FROM hierarchie ORDER BY niveau;
```

## CREATE VIEW
```sql
-- Créer
CREATE VIEW employes_paris AS
SELECT nom, age, salaire
FROM employes
WHERE ville = 'Paris';

-- Utiliser
SELECT * FROM employes_paris;

-- Modifier
CREATE OR REPLACE VIEW employes_paris AS
SELECT nom, age, salaire, date_embauche
FROM employes WHERE ville = 'Paris';

-- Supprimer
DROP VIEW IF EXISTS employes_paris;
```

## CTE vs Sous-requête vs Vue
| Outil | Durée de vie | Usage |
|---|---|---|
| Sous-requête | Une expression | Logique simple |
| CTE | Une requête | Logique complexe, multi-étapes |
| Vue | Permanente | Réutilisation, partage |

## Pipeline analytique avec CTEs
```sql
WITH
donnees_propres AS (
    SELECT * FROM ventes
    WHERE montant > 0 AND date_vente IS NOT NULL
),
stats AS (
    SELECT client_id,
           SUM(montant) AS total,
           COUNT(*)     AS nb_achats
    FROM donnees_propres
    GROUP BY client_id
),
ranked AS (
    SELECT *,
        RANK() OVER (ORDER BY total DESC) AS rang
    FROM stats
)
SELECT * FROM ranked WHERE rang <= 10;
```
