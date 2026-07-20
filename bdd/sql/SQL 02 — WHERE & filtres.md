#sql #where #filtres #conditions

## Opérateurs de comparaison
```sql
WHERE age = 25
WHERE age != 25    -- ou <>
WHERE age > 25
WHERE age >= 25
WHERE age < 30
WHERE age <= 30
```

## AND, OR, NOT
```sql
WHERE ville = 'Paris' AND salaire > 3000
WHERE ville = 'Paris' OR ville = 'Lyon'
WHERE NOT ville = 'Paris'
```

> [!warning] Priorité des opérateurs
> `AND` évalué avant `OR` → toujours utiliser des parenthèses !
> ```sql
> WHERE (ville = 'Paris' OR ville = 'Lyon') AND salaire > 4000
> ```

## IN / NOT IN
```sql
WHERE ville IN ('Paris', 'Lyon', 'Bordeaux')
WHERE ville NOT IN ('Paris', 'Lyon')
```

## BETWEEN
```sql
WHERE age BETWEEN 25 AND 32      -- bornes INCLUSES
WHERE salaire NOT BETWEEN 3000 AND 4000
```

## LIKE
```sql
WHERE nom LIKE 'A%'      -- commence par A
WHERE nom LIKE '%a'      -- finit par a
WHERE nom LIKE '%li%'    -- contient "li"
WHERE nom LIKE '_ob'     -- 1 char + "ob"
WHERE nom NOT LIKE 'A%'
```

| Wildcard | Signification |
|---|---|
| `%` | N'importe quelle séquence |
| `_` | Exactement 1 caractère |

## IS NULL / IS NOT NULL
```sql
WHERE salaire IS NULL       -- ✅
WHERE salaire IS NOT NULL   -- ✅
WHERE salaire = NULL        -- ❌ ne fonctionne jamais !
```

> [!warning] NULL est spécial
> `NULL = NULL` retourne NULL, pas TRUE. Toujours utiliser `IS NULL` / `IS NOT NULL`

## Guillemets
```sql
WHERE ville = 'Paris'     -- ✅ valeurs texte → guillemets simples
WHERE ville = "Paris"     -- ❌ guillemets doubles = noms de colonnes
```
