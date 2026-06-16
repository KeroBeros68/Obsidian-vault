#sql #texte #dates #fonctions

## Fonctions texte
```sql
UPPER(nom)                        -- MAJUSCULES
LOWER(nom)                        -- minuscules
LENGTH(nom)                       -- longueur
TRIM('  Alice  ')                 -- 'Alice'
LTRIM / RTRIM                     -- gauche / droite
SUBSTRING(nom, 1, 3)              -- 3 premiers chars
LEFT(nom, 3)                      -- 3 premiers chars
RIGHT(nom, 3)                     -- 3 derniers chars
REPLACE(nom, 'a', 'A')            -- remplacer
POSITION('li' IN 'Alice')         -- position (PostgreSQL)
INSTR('Alice', 'li')              -- position (MySQL)
LPAD('42', 5, '0')                -- '00042'
RPAD('Alice', 10, '.')            -- 'Alice.....'
REVERSE('Alice')                  -- 'ecilA'
```

## Concaténation
```sql
CONCAT(prenom, ' ', nom)          -- universel
prenom || ' ' || nom              -- PostgreSQL, SQLite
prenom + ' ' + nom                -- SQL Server
CONCAT_WS(', ', ville, pays)      -- ignore les NULL ✅
```

## Date actuelle
```sql
CURRENT_DATE        -- date du jour
CURRENT_TIMESTAMP   -- date + heure
NOW()               -- PostgreSQL, MySQL
GETDATE()           -- SQL Server
```

## Extraire des parties de date
```sql
EXTRACT(YEAR  FROM date_col)   -- standard SQL
EXTRACT(MONTH FROM date_col)
EXTRACT(DAY   FROM date_col)
YEAR(date_col)                 -- MySQL, SQL Server
MONTH(date_col)
DAY(date_col)
```

## Calculs sur les dates
```sql
-- PostgreSQL
date_col + INTERVAL '1 year'
date_col - INTERVAL '3 months'
AGE(date1, date2)              -- différence

-- MySQL
DATE_ADD(date_col, INTERVAL 1 YEAR)
DATEDIFF(date1, date2)         -- différence en jours

-- SQL Server
DATEDIFF(day, date1, date2)
```

## Formater les dates
```sql
TO_CHAR(date_col, 'DD/MM/YYYY')    -- PostgreSQL
DATE_FORMAT(date_col, '%d/%m/%Y')  -- MySQL
FORMAT(date_col, 'dd/MM/yyyy')     -- SQL Server
```

## Convertir texte → date
```sql
CAST('2023-01-15' AS DATE)
'2023-01-15'::DATE                     -- PostgreSQL
STR_TO_DATE('15/01/2023', '%d/%m/%Y')  -- MySQL
```

## Ventes par mois — pattern courant
```sql
SELECT
    EXTRACT(YEAR  FROM date_vente) AS annee,
    EXTRACT(MONTH FROM date_vente) AS mois,
    COUNT(*)     AS nb,
    SUM(montant) AS total
FROM ventes
GROUP BY 1, 2
ORDER BY 1, 2;
```
