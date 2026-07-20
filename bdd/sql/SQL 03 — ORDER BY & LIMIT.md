#sql #order-by #limit #tri #pagination

## ORDER BY
```sql
ORDER BY salaire           -- ASC par défaut (croissant)
ORDER BY salaire ASC       -- croissant explicite
ORDER BY salaire DESC      -- décroissant
```

## Tri multi-colonnes
```sql
ORDER BY ville ASC, salaire DESC
-- Chaque colonne a son propre ASC/DESC
```

## LIMIT / OFFSET
```sql
LIMIT 10                   -- 10 premières lignes
LIMIT 10 OFFSET 20         -- lignes 21-30 (page 3)

-- Page n avec k résultats par page :
LIMIT k OFFSET (n-1) * k
```

## Selon le SGBD
| SGBD | Syntaxe |
|---|---|
| MySQL, PostgreSQL, SQLite | `LIMIT n` |
| SQL Server | `SELECT TOP n` |
| Oracle | `WHERE ROWNUM <= n` |

## Combinaison complète
```sql
SELECT nom, ville, salaire
FROM employes
WHERE ville = 'Paris'
ORDER BY salaire DESC
LIMIT 3;
-- → 3 employés de Paris les mieux payés
```

## NULL dans le tri
```sql
ORDER BY salaire ASC NULLS LAST    -- PostgreSQL
ORDER BY salaire ASC NULLS FIRST   -- PostgreSQL
-- MySQL : NULL en premier par défaut en ASC
```
