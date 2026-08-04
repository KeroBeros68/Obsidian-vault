#sql #fonctions #count #sum #avg #case

## Fonctions d'agrégation
```sql
SELECT
    COUNT(*)           AS nb_total,      -- toutes les lignes
    COUNT(salaire)     AS nb_avec_sal,   -- exclut les NULL
    COUNT(DISTINCT v)  AS nb_villes,     -- valeurs uniques
    SUM(salaire)       AS total,
    AVG(salaire)       AS moyenne,
    MIN(salaire)       AS minimum,
    MAX(salaire)       AS maximum
FROM employes;
```

> [!tip] COUNT(*) vs COUNT(col)
> `COUNT(*)` → toutes les lignes
> `COUNT(col)` → lignes non-NULL seulement

## Fonctions mathématiques
```sql
ROUND(3.14159, 2)    -- 3.14
ABS(-42)             -- 42
CEIL(4.2)            -- 5
FLOOR(4.8)           -- 4
MOD(10, 3)           -- 1
POWER(2, 8)          -- 256
```

## Fonctions texte
```sql
UPPER(nom)                    -- MAJUSCULES
LOWER(nom)                    -- minuscules
LENGTH(nom)                   -- longueur
TRIM('  Alice  ')             -- 'Alice'
SUBSTRING(nom, 1, 3)          -- 3 premiers chars
REPLACE(nom, 'a', 'A')        -- remplacer
CONCAT(prenom, ' ', nom)      -- concaténer
LEFT(nom, 3)                  -- 3 premiers chars
RIGHT(nom, 3)                 -- 3 derniers chars
```

## CASE WHEN — if/else SQL
```sql
CASE
    WHEN salaire >= 5000 THEN 'Haut'
    WHEN salaire >= 3500 THEN 'Moyen'
    ELSE                      'Bas'
END AS niveau

-- Sur valeur exacte
CASE ville
    WHEN 'Paris' THEN 75
    WHEN 'Lyon'  THEN 69
    ELSE 0
END AS code_dep
```

## COALESCE — gérer les NULL
```sql
COALESCE(bonus, salaire, 0)
-- retourne la 1ère valeur non-NULL
-- équivalent : fillna() en Pandas
```

## CAST — convertir les types
```sql
CAST(salaire AS VARCHAR)
CAST('42' AS INTEGER)
salaire::VARCHAR              -- PostgreSQL
```
