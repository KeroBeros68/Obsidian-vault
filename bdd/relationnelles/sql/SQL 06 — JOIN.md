#sql #join #inner #left #right #full

## Syntaxe de base
```sql
SELECT e.nom, d.nom_dep
FROM employes e
INNER JOIN departements d ON e.dep_id = d.id;
```

## Types de JOIN

### INNER JOIN — intersection
```sql
FROM employes e
INNER JOIN departements d ON e.dep_id = d.id
-- Seulement les lignes avec correspondance dans les deux tables
```

### LEFT JOIN — tout à gauche
```sql
FROM employes e
LEFT JOIN departements d ON e.dep_id = d.id
-- Tous les employés + NULL si pas de département
```

### RIGHT JOIN — tout à droite
```sql
FROM employes e
RIGHT JOIN departements d ON e.dep_id = d.id
-- Tous les départements + NULL si pas d'employé
```

### FULL OUTER JOIN — tout
```sql
FROM employes e
FULL OUTER JOIN departements d ON e.dep_id = d.id
-- Toutes les lignes des deux tables
```

## Visualisation
```
     gauche    droite
INNER  ∩          ∩   intersection
LEFT   ■          ∩   tout gauche
RIGHT  ∩          ■   tout droite
FULL   ■          ■   tout
```

> [!tip] RIGHT JOIN rarement utilisé
> Préférer inverser les tables et utiliser LEFT JOIN

## JOIN sur 3 tables
```sql
FROM commandes c
INNER JOIN clients  cl ON c.client_id  = cl.id
INNER JOIN produits p  ON c.produit_id = p.id
```

## Lignes sans correspondance
```sql
-- Employés SANS département
SELECT e.nom
FROM employes e
LEFT JOIN departements d ON e.dep_id = d.id
WHERE d.id IS NULL;    -- ← pattern clé !
```

## CROSS JOIN — produit cartésien
```sql
FROM employes e CROSS JOIN departements d
-- n × m lignes (rarement utile)
```
