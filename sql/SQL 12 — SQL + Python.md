#sql #python #pandas #sqlalchemy #sqlite

## Connexion avec sqlite3 — bibliothèque standard

python

```python
import sqlite3
import pandas as pd

# Connexion à une base SQLite
conn = sqlite3.connect("ma_base.db")   # crée si n'existe pas
conn = sqlite3.connect(":memory:")     # base en mémoire (tests)

# Exécuter une requête simple
cursor = conn.cursor()
cursor.execute("SELECT * FROM employes WHERE ville = 'Paris'")
resultats = cursor.fetchall()           # liste de tuples
colnames = [desc[0] for desc in cursor.description]

# Fermer la connexion
conn.close()
```

## pd.read_sql — lire SQL dans un DataFrame

python

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect("ma_base.db")

# Requête SQL → DataFrame directement
df = pd.read_sql(
    "SELECT * FROM employes WHERE salaire > 3000",
    conn
)

# Avec paramètres (évite les injections SQL)
df = pd.read_sql(
    "SELECT * FROM employes WHERE ville = ?",
    conn,
    params=("Paris",)
)

# Requête complexe avec CTE
query = """
    WITH stats AS (
        SELECT ville, AVG(salaire) AS moy
        FROM employes
        GROUP BY ville
    )
    SELECT * FROM stats WHERE moy > 3500
"""
df = pd.read_sql(query, conn)
conn.close()
```

## df.to_sql — écrire un DataFrame en SQL

python

```python
# DataFrame → table SQL
df.to_sql(
    name      = "employes_new",
    con       = conn,
    if_exists = "replace",   # "replace", "append", "fail"
    index     = False        # ne pas écrire l'index Pandas
)

# if_exists :
# "fail"    → erreur si la table existe déjà
# "replace" → supprime et recrée la table
# "append"  → ajoute les lignes à la table existante
```

## SQLAlchemy — connexion universelle

python

```python
from sqlalchemy import create_engine
import pandas as pd

# Chaînes de connexion selon le SGBD
engine = create_engine("sqlite:///ma_base.db")
engine = create_engine("postgresql://user:pass@localhost/db")
engine = create_engine("mysql+pymysql://user:pass@localhost/db")
engine = create_engine("mssql+pyodbc://user:pass@server/db")

# Lire
with engine.connect() as conn:
    df = pd.read_sql("SELECT * FROM employes", conn)

# Écrire
df.to_sql("employes", engine, if_exists="replace", index=False)
```

## Paramètres dans les requêtes — éviter les injections SQL

python

```python
# ❌ DANGEREUX — injection SQL possible !
ville = "Paris'; DROP TABLE employes; --"
query = f"SELECT * FROM employes WHERE ville = '{ville}'"

# ✅ SÉCURISÉ — paramètres liés
# sqlite3
cursor.execute("SELECT * FROM employes WHERE ville = ?", (ville,))

# SQLAlchemy
from sqlalchemy import text
with engine.connect() as conn:
    result = conn.execute(
        text("SELECT * FROM employes WHERE ville = :ville"),
        {"ville": "Paris"}
    )
```

## Workflow complet — analyse avec SQL + Pandas

python

```python
import sqlite3
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# 1. Connexion
conn = sqlite3.connect("ventes.db")

# 2. Chargement avec SQL
df_ventes = pd.read_sql("""
    SELECT
        v.date_vente,
        v.montant,
        c.ville,
        p.categorie
    FROM ventes v
    JOIN clients  c ON v.client_id  = c.id
    JOIN produits p ON v.produit_id = p.id
    WHERE v.montant > 0
      AND v.date_vente >= '2024-01-01'
""", conn)

# 3. Transformation avec Pandas
df_ventes["date_vente"] = pd.to_datetime(df_ventes["date_vente"])
df_ventes["mois"] = df_ventes["date_vente"].dt.to_period("M")

# 4. Analyse
stats = df_ventes.groupby(["ville","categorie"])["montant"].agg([
    "sum","mean","count"
]).reset_index()

# 5. Visualisation
fig, ax = plt.subplots(figsize=(10, 6))
sns.barplot(data=stats, x="ville", y="sum",
            hue="categorie", ax=ax)
ax.set_title("Ventes par ville et catégorie")
plt.show()

# 6. Sauvegarder les résultats
stats.to_sql("stats_ventes", conn, if_exists="replace", index=False)
conn.close()
```

## Créer une base SQLite depuis Python

python

```python
conn = sqlite3.connect("ma_base.db")
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS employes (
        id      INTEGER PRIMARY KEY AUTOINCREMENT,
        nom     TEXT    NOT NULL,
        age     INTEGER,
        ville   TEXT,
        salaire REAL
    )
""")

cursor.executemany(
    "INSERT INTO employes (nom, age, ville, salaire) VALUES (?,?,?,?)",
    [
        ("Alice", 25, "Paris",    3000),
        ("Bob",   32, "Lyon",     4500),
        ("Clara", 28, "Paris",    3800),
    ]
)

conn.commit()
conn.close()
```

## Quand utiliser SQL vs Pandas ?

|Tâche|SQL|Pandas|
|---|---|---|
|Lire des données filtrées|✅ Idéal|Possible|
|Jointures sur grandes tables|✅ Plus rapide|✅ OK|
|Transformations complexes|Possible|✅ Idéal|
|Visualisation|❌|✅ Idéal|
|Partager des résultats|✅ Vue|✅ to_sql|
|Données > RAM|✅|Chunking|

> [!tip] Bonne pratique Faire le **filtrage, les jointures et les agrégations en SQL** — les données arrivent déjà propres dans Pandas. Faire la **transformation, l'analyse et la visualisation en Pandas/Matplotlib**.