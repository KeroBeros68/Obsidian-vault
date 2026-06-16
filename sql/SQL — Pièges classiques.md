#sql #pièges #erreurs #debugging

## Piège 1 — WHERE salaire = NULL
```sql
WHERE salaire = NULL     -- ❌ retourne toujours 0 ligne !
WHERE salaire IS NULL    -- ✅
WHERE salaire IS NOT NULL -- ✅
```

## Piège 2 — NOT IN avec NULL dans la liste
```sql
WHERE id NOT IN (1, 2, NULL)   -- ❌ retourne 0 ligne !
-- NULL contamine tout NOT IN

-- Solution :
WHERE NOT EXISTS (
    SELECT 1 FROM table2
    WHERE table2.id = table1.id
)
```

## Piège 3 — Guillemets simples vs doubles
```sql
WHERE ville = "Paris"    -- ❌ guillemets doubles = nom de colonne
WHERE ville = 'Paris'    -- ✅
```

## Piège 4 — Colonne non agrégée dans GROUP BY
```sql
-- ❌ ERREUR
SELECT ville, nom, AVG(salaire)
FROM employes GROUP BY ville;

-- ✅
SELECT ville, AVG(salaire)
FROM employes GROUP BY ville;
```

## Piège 5 — AVG dans WHERE au lieu de HAVING
```sql
WHERE AVG(salaire) > 4000    -- ❌ AVG interdit dans WHERE

GROUP BY ville
HAVING AVG(salaire) > 4000   -- ✅
```

## Piège 6 — BETWEEN exclut les bornes... NON !
```sql
WHERE age BETWEEN 25 AND 30
-- Les bornes sont INCLUSES : équivalent à age >= 25 AND age <= 30
```

## Piège 7 — ORDER BY après LIMIT
```sql
SELECT * FROM employes LIMIT 3 ORDER BY salaire;  -- ❌ erreur syntaxe
SELECT * FROM employes ORDER BY salaire LIMIT 3;  -- ✅
```

## Piège 8 — Window function dans WHERE
```sql
-- ❌ ERREUR — window function évaluée après WHERE
WHERE RANK() OVER (ORDER BY salaire DESC) <= 3

-- ✅ Sous-requête obligatoire
SELECT * FROM (
    SELECT *, RANK() OVER (ORDER BY salaire DESC) AS rang
    FROM employes
) r WHERE rang <= 3;
```

## Piège 9 — Alias non utilisable dans WHERE
```sql
SELECT salaire * 12 AS annuel
FROM employes
WHERE annuel > 50000;    -- ❌ alias pas encore calculé !

-- ✅
WHERE salaire * 12 > 50000    -- répéter l'expression
```

## Piège 10 — Table dérivée sans alias
```sql
SELECT * FROM (
    SELECT ville, AVG(salaire) AS moy
    FROM employes GROUP BY ville
);                          -- ❌ alias manquant !

SELECT * FROM (
    SELECT ville, AVG(salaire) AS moy
    FROM employes GROUP BY ville
) AS stats_villes;          -- ✅
```

## Piège 11 — COUNT(*) vs COUNT(col)
```sql
-- Sur une table avec des NULL dans salaire :
COUNT(*)        -- compte TOUTES les lignes
COUNT(salaire)  -- exclut les lignes où salaire est NULL
-- Les deux peuvent donner des résultats très différents !
```

## Piège 12 — LIKE insensible à la casse (selon SGBD)
```sql
WHERE nom LIKE 'alice%'
-- MySQL → trouve 'Alice' ✅ (insensible par défaut)
-- PostgreSQL → ne trouve pas 'Alice' ❌

-- PostgreSQL insensible à la casse :
WHERE nom ILIKE 'alice%'   -- ✅
```

## Récapitulatif rapide
| Piège | Solution |
|---|---|
| `= NULL` | Utiliser `IS NULL` |
| `NOT IN` avec NULL | Utiliser `NOT EXISTS` |
| Guillemets doubles | Guillemets simples pour les valeurs |
| Colonne non agrégée | Ajouter au GROUP BY ou agréger |
| AVG dans WHERE | Utiliser HAVING |
| BETWEEN exclut bornes | Non — les bornes sont INCLUSES |
| ORDER BY après LIMIT | ORDER BY avant LIMIT |
| Window function dans WHERE | Sous-requête obligatoire |
| Alias dans WHERE | Répéter l'expression ou CTE |
| Table dérivée sans alias | Alias obligatoire après `)` |
| COUNT(*) vs COUNT(col) | `COUNT(*)` = tout, `COUNT(col)` = sans NULL |
| LIKE case-sensitive | `ILIKE` sur PostgreSQL |
