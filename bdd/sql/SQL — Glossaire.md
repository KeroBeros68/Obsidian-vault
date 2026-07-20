#sql #glossaire #référence

| Terme | Définition |
|---|---|
| **SGBD** | Système de Gestion de Base de Données (MySQL, PostgreSQL, SQLite, SQL Server...) |
| **Table** | Structure de données en lignes et colonnes |
| **Clé primaire** | Identifiant unique de chaque ligne (`id`) |
| **Clé étrangère** | Colonne qui référence la clé primaire d'une autre table |
| **NULL** | Absence de valeur — différent de 0 ou chaîne vide |
| **SELECT** | Clause pour choisir les colonnes à afficher |
| **FROM** | Clause pour spécifier la table source |
| **WHERE** | Filtre les lignes avant le regroupement |
| **GROUP BY** | Regroupe les lignes par valeur de colonne |
| **HAVING** | Filtre les groupes après GROUP BY |
| **ORDER BY** | Trie les résultats (ASC par défaut) |
| **LIMIT** | Limite le nombre de lignes retournées |
| **OFFSET** | Saute n lignes — utilisé pour la pagination |
| **DISTINCT** | Élimine les doublons dans le résultat |
| **AS** | Crée un alias pour une colonne ou une table |
| **INNER JOIN** | Retourne uniquement les lignes avec correspondance |
| **LEFT JOIN** | Retourne toutes les lignes de gauche + correspondances |
| **FULL OUTER JOIN** | Retourne toutes les lignes des deux tables |
| **ON** | Condition de jointure entre deux tables |
| **Sous-requête** | Requête imbriquée dans une autre requête |
| **EXISTS** | Vérifie l'existence d'une ligne dans une sous-requête |
| **IN** | Vérifie si une valeur est dans une liste |
| **BETWEEN** | Filtre dans un intervalle (bornes incluses) |
| **LIKE** | Recherche par motif avec % et _ |
| **CASE WHEN** | Condition if/else en SQL |
| **COALESCE** | Retourne la première valeur non-NULL |
| **CAST** | Convertit un type en un autre |
| **Window function** | Calcul sur un ensemble de lignes sans réduire le nombre de lignes |
| **OVER** | Clause qui définit la fenêtre d'une window function |
| **PARTITION BY** | Divise les données en groupes pour une window function |
| **ROW_NUMBER** | Numéro unique par ligne dans une fenêtre |
| **RANK** | Rang avec saut en cas d'ex-æquo |
| **DENSE_RANK** | Rang sans saut en cas d'ex-æquo |
| **LAG / LEAD** | Accède aux valeurs des lignes précédentes/suivantes |
| **CTE** | Common Table Expression — requête nommée temporaire avec WITH |
| **Vue** | Requête sauvegardée comme table virtuelle permanente |
| **DDL** | Data Definition Language — CREATE, ALTER, DROP |
| **DML** | Data Manipulation Language — SELECT, INSERT, UPDATE, DELETE |
