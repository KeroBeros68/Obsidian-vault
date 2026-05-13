#matplotlib #figure #axes #architecture

## Import
```python
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
```

## Deux interfaces
```python
# Interface pyplot — rapide, exploration
plt.plot(x, y)
plt.title("Titre")
plt.show()

# Interface OO — recommandée pour tout le reste
fig, ax = plt.subplots()
ax.plot(x, y)
ax.set_title("Titre")
plt.show()
```

> [!tip] Toujours utiliser l'interface OO
> `fig, ax = plt.subplots()` → contrôle total, subplots, publication

## Hiérarchie des objets
```
Figure  ← cadre global ("la feuille")
  └── Axes  ← zone de dessin (le graphique)
        ├── Axis   ← axes X et Y
        ├── Title  ← titre
        ├── Labels ← étiquettes
        └── Artists ← lignes, barres, points...
```

## Anatomie d'un graphique
```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, y, label="Série A")
ax.set_title("Titre", fontsize=14, fontweight="bold")
ax.set_xlabel("X", fontsize=12)
ax.set_ylabel("Y", fontsize=12)
ax.set_xlim(0, 10)
ax.set_ylim(-1, 1)
ax.set_xticks([0, 5, 10])
ax.set_xticklabels(["Jan","Juin","Déc"])
ax.grid(True, alpha=0.3)
ax.legend(loc="upper right")
plt.tight_layout()
plt.show()
```

## Cycle de vie
```python
fig, ax = plt.subplots()   # 1. Créer
ax.plot(x, y)              # 2. Dessiner
ax.set_title("...")        # 3. Personnaliser
fig.savefig("out.png",     # 4. Sauvegarder (AVANT show !)
    dpi=300, bbox_inches="tight")
plt.show()                 # 5. Afficher
plt.close(fig)             # 6. Fermer (libère mémoire)
```
