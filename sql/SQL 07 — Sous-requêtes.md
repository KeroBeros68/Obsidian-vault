#sql #sous-requêtes #subquery #exists #in

## Sous-requête scalaire — retourne une valeur
```sql
SELECT nom, salaire
FROM employes
WHERE salaire > (SELECT AVG(salaire) FROM employes);
```

## Sous-requête dans le SELECT
```sql
SELECT nom, salaire,
    (SELECT MAX(salaire) FROM employes) AS max_global,
    salaire - (SELECT AVG(salaire) FROM employes) AS ecart
FROM employes;
```

## IN — liste de valeurs
```sql
WHERE dep_id IN (
    SELECT id FROM departements
    WHERE budget > 50000
)
```

> [!warning] NOT IN et NULL
> Si la sous-requête retourne un NULL → NOT IN retourne toujours FALSE !
> Utiliser NOT EXISTS à la place.

## EXISTS / NOT EXISTS
```sql
-- Plus performant que IN sur grandes tables
WHERE EXISTS (
    SELECT 1 FROM commandes c
    WHERE c.employe_id = e.id
)
-- S'arrête dès la 1ère correspondance trouvée
```

## Table dérivée — sous-requête dans FROM
```sql
SELECT ville, salaire_moyen
FROM (
    SELECT ville, AVG(salaire) AS salaire_moyen
    FROM employes
    GROUP BY ville
) AS stats_villes           -- alias OBLIGATOIRE !
WHERE salaire_moyen > 3500;
```

## Sous-requête corrélée
```sql
-- Employés gagnant plus que la moyenne de leur ville
SELECT e.nom, e.ville, e.salaire
FROM employes e
WHERE e.salaire > (
    SELECT AVG(e2.salaire)
    FROM employes e2
    WHERE e2.ville = e.ville   -- référence la ligne externe
);
```

## Quand utiliser quoi ?
| Outil | Usage |
|---|---|
| JOIN | Colonnes des deux tables dans le résultat |
| Sous-requête IN | Filtrer selon une autre table |
| EXISTS | Vérifier l'existence sans récupérer de données |
| Table dérivée | Filtrer sur un résultat agrégé |
