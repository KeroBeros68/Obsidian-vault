#python #pandas #glossaire #référence

|Terme|Définition|
|---|---|
|**Series**|Tableau 1D avec index — colonne d'un DataFrame|
|**DataFrame**|Tableau 2D avec index et colonnes nommées|
|**index**|Étiquettes des lignes d'un DataFrame ou d'une Series|
|**dtype**|Type de données d'une colonne (int64, float64, object, category...)|
|**NaN**|Not a Number — valeur manquante en Pandas|
|**loc**|Sélection par étiquette (label)|
|**iloc**|Sélection par position (integer)|
|**groupby**|Opération Split-Apply-Combine — comme GROUP BY en SQL|
|**agg**|Agrégation qui réduit un groupe à une valeur|
|**transform**|Agrégation qui conserve la shape originale|
|**merge**|Jointure entre deux DataFrames sur une clé commune|
|**concat**|Empilement de DataFrames en lignes ou colonnes|
|**one-hot encoding**|Transformation d'une variable catégorielle en colonnes binaires|
|**label encoding**|Transformation d'une variable ordinale en entiers|
|**data leakage**|Contamination du train par des infos du test — biais majeur en ML|
|**StandardScaler**|Normalisation Z-score : moyenne=0, écart-type=1|
|**MinMaxScaler**|Normalisation entre 0 et 1|
|**category**|dtype Pandas pour les colonnes avec peu de valeurs uniques — économise la mémoire|
|**chunksize**|Lecture d'un fichier par morceaux pour économiser la mémoire|
|**inplace**|Modifie le DataFrame directement sans retourner de copie|
|**EDA**|Exploratory Data Analysis — exploration systématique d'un dataset|
|**IQR**|Inter-Quartile Range = Q3 - Q1 — mesure de dispersion robuste|
|**ffill**|Forward fill — propage la dernière valeur connue vers le bas|
|**bfill**|Backward fill — propage la prochaine valeur connue vers le haut|
|**query**|Filtrage avec une syntaxe SQL-like lisible|
|**assign**|Ajoute des colonnes calculées de façon chaînable|
|**pivot_table**|Tableau croisé dynamique — équivalent Excel|
|**crosstab**|Tableau de contingence entre deux variables catégorielles|
