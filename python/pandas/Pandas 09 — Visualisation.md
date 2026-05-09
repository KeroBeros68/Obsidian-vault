#pandas #matplotlib #seaborn #visualisation #graphiques

## Pandas plot — syntaxe de base

python

```python
import matplotlib.pyplot as plt

df["col"].plot(kind="hist")
df.plot(kind="bar")
df.plot(kind="scatter", x="age", y="salaire")
df.plot(kind="box")

plt.title("Titre")
plt.xlabel("X")
plt.ylabel("Y")
plt.tight_layout()
plt.savefig("graph.png", dpi=300, bbox_inches="tight")  # AVANT show !
plt.show()
```

> [!warning] savefig AVANT show `plt.show()` vide le buffer → image vide si on sauvegarde après !

## Choisir le bon graphique

|Besoin|Graphique|
|---|---|
|Distribution variable continue|histogramme, boxplot|
|Comparer des catégories|barplot|
|Relation entre 2 variables|scatter|
|Évolution temporelle|line plot|
|Corrélations|heatmap|
|Distribution par groupe|boxplot, violinplot|

## Seaborn — graphiques statistiques

python

```python
import seaborn as sns

sns.histplot(df["col"], kde=True)
sns.scatterplot(data=df, x="age", y="salaire", hue="ville")
sns.boxplot(data=df, x="ville", y="salaire")
sns.violinplot(data=df, x="ville", y="salaire")
sns.heatmap(df.corr(), annot=True, cmap="coolwarm", fmt=".2f")
sns.pairplot(df, hue="ville")
```

## Plusieurs graphiques — subplots

python

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
df["age"].hist(ax=axes[0])
df["salaire"].hist(ax=axes[1])
plt.tight_layout()
plt.show()
```
