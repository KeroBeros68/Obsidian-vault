#sql #group-by #having #agrégations

## GROUP BY — regrouper
```sql
SELECT ville, AVG(salaire) AS moy
FROM employes
GROUP BY ville;
-- → une ligne par ville
```

> [!warning] Règle fondamentale
> Dans un SELECT avec GROUP BY, chaque colonne doit soit être dans le `GROUP BY`, soit être dans une fonction d'agrégation.

```sql
-- ❌ ERREUR — nom non agrégé
SELECT ville, nom, AVG(salaire) FROM employes GROUP BY ville;

-- ✅ CORRECT
SELECT ville, AVG(salaire) FROM employes GROUP BY ville;
```

## Plusieurs agrégations
```sql
SELECT ville,
    COUNT(*)        AS nb,
    AVG(salaire)    AS moy,
    MIN(salaire)    AS min_sal,
    MAX(salaire)    AS max_sal,
    SUM(salaire)    AS total
FROM employes
GROUP BY ville
ORDER BY moy DESC;
```

## HAVING — filtrer les groupes
```sql
SELECT ville, COUNT(*) AS nb
FROM employes
GROUP BY ville
HAVING COUNT(*) > 1;   -- villes avec > 1 employé
```

## WHERE vs HAVING
| | WHERE | HAVING |
|---|---|---|
| Quand | Avant GROUP BY | Après GROUP BY |
| Sur | Les lignes | Les groupes |
| Agrégations | ❌ Interdit | ✅ Obligatoire |

## Ordre d'exécution SQL
```
1. FROM      → quelle table
2. WHERE     → filtre lignes
3. GROUP BY  → crée groupes
4. HAVING    → filtre groupes
5. SELECT    → choisit colonnes
6. ORDER BY  → trie
7. LIMIT     → limite
```

## Exemple complet
```sql
SELECT ville, AVG(salaire) AS moy
FROM employes
WHERE age > 25
GROUP BY ville
HAVING AVG(salaire) > 3500
ORDER BY moy DESC;
```
