#sql #window-functions #over #partition-by #rank

## Syntaxe de base
```sql
fonction() OVER (
    PARTITION BY colonne   -- groupes (optionnel)
    ORDER BY    colonne    -- ordre (optionnel)
)
```

## Différence avec GROUP BY
```sql
-- GROUP BY → réduit (1 ligne par groupe)
SELECT ville, AVG(salaire) FROM employes GROUP BY ville;

-- Window function → conserve toutes les lignes
SELECT nom, ville, salaire,
    AVG(salaire) OVER (PARTITION BY ville) AS moy_ville
FROM employes;
```

## Agrégations en fenêtre
```sql
SELECT nom, ville, salaire,
    AVG(salaire) OVER (PARTITION BY ville) AS moy_ville,
    MAX(salaire) OVER (PARTITION BY ville) AS max_ville,
    COUNT(*)     OVER (PARTITION BY ville) AS nb_ville,
    AVG(salaire) OVER ()                   AS moy_globale
FROM employes;
```

## Fonctions de rang
```sql
ROW_NUMBER() OVER (ORDER BY salaire DESC)  -- toujours unique
RANK()       OVER (ORDER BY salaire DESC)  -- ex-æquo → saute rangs (1,1,3)
DENSE_RANK() OVER (ORDER BY salaire DESC)  -- ex-æquo → sans saut (1,1,2)
NTILE(4)     OVER (ORDER BY salaire)       -- divise en 4 groupes égaux
```

> [!tip] Différence des rangs
> `ROW_NUMBER` → 1,2,3,4,5
> `RANK`       → 1,2,2,4,5 (saute 3)
> `DENSE_RANK` → 1,2,2,3,4 (ne saute pas)

## Rang par partition
```sql
SELECT nom, ville, salaire,
    RANK() OVER (
        PARTITION BY ville
        ORDER BY salaire DESC
    ) AS rang_dans_ville
FROM employes;
```

## Top N par groupe — pattern classique
```sql
SELECT *
FROM (
    SELECT nom, ville, salaire,
        RANK() OVER (
            PARTITION BY ville
            ORDER BY salaire DESC
        ) AS rang
    FROM employes
) ranked
WHERE rang <= 2;
```

## LAG / LEAD — lignes voisines
```sql
LAG(ventes, 1)  OVER (ORDER BY mois)  -- ligne précédente
LEAD(ventes, 1) OVER (ORDER BY mois)  -- ligne suivante

-- Évolution mois/mois
ventes - LAG(ventes, 1) OVER (ORDER BY mois) AS evolution
```

## Cumul et moyenne mobile
```sql
SUM(ventes) OVER (ORDER BY mois) AS cumul

AVG(ventes) OVER (
    ORDER BY mois
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) AS moyenne_mobile_3mois
```

## Autres fonctions
```sql
FIRST_VALUE(col) OVER (...)    -- 1ère valeur de la fenêtre
LAST_VALUE(col)  OVER (...)    -- dernière valeur
NTH_VALUE(col, n) OVER (...)   -- nème valeur
```
