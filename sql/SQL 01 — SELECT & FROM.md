#sql #select #from #bases

## Anatomie d'une requête
```sql
SELECT colonne1, colonne2   -- quoi afficher
FROM table;                 -- depuis quelle table
```

## Sélection
```sql
SELECT *                    -- toutes les colonnes
SELECT nom, age, ville      -- colonnes spécifiques
SELECT DISTINCT ville       -- valeurs uniques
```

## Alias
```sql
SELECT nom AS "Nom complet",
       salaire AS salaire_mensuel
FROM employes;
```

## Colonnes calculées
```sql
SELECT nom,
       salaire * 12    AS salaire_annuel,
       salaire * 0.75  AS salaire_net
FROM employes;
```

## Ordre obligatoire des clauses
```sql
SELECT   -- 1
FROM     -- 2
WHERE    -- 3
GROUP BY -- 4
HAVING   -- 5
ORDER BY -- 6
LIMIT    -- 7
```

> [!warning] L'ordre est strict
> Toute déviation → erreur de syntaxe

## Commentaires
```sql
-- Commentaire ligne
/* Commentaire
   multi-lignes */
```

> [!tip] Convention
> MAJUSCULES pour les mots-clés SQL, minuscules pour les noms de tables et colonnes
