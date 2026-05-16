#matplotlib #seaborn #glossaire #référence

| Terme | Définition |
|---|---|
| **Figure** | Cadre global d'un graphique Matplotlib — la "feuille de papier" |
| **Axes** | Zone de dessin à l'intérieur d'une Figure — contient les données, titres, labels |
| **Axis** | Un axe individuel (X ou Y) d'un Axes |
| **Artist** | Tout objet dessiné dans un Axes (ligne, barre, point, texte...) |
| **rcParams** | Paramètres de configuration globaux de Matplotlib |
| **DPI** | Dots Per Inch — résolution d'une image (72=écran, 300=impression) |
| **tight_layout** | Ajustement automatique des espacements pour éviter les chevauchements |
| **constrained_layout** | Version moderne de tight_layout, activé à la création de la figure |
| **subplot_mosaic** | Création de layouts asymétriques via une carte ASCII |
| **GridSpec** | Création de grilles avec contrôle fin des proportions |
| **twinx** | Crée un 2e axe Y partageant le même axe X |
| **twiny** | Crée un 2e axe X partageant le même axe Y |
| **axes.flat** | Itérateur 1D sur tous les Axes d'une grille 2D |
| **hue** | Paramètre Seaborn qui colorie par catégorie et ajoute une légende |
| **axes-level** | Fonction Seaborn qui s'intègre dans une figure Matplotlib (`ax=`) |
| **figure-level** | Fonction Seaborn qui crée sa propre figure (`col=`, `row=`) |
| **FacetGrid** | Grille de graphiques Seaborn créée manuellement par facettes |
| **kde** | Kernel Density Estimate — courbe de densité lissée |
| **despine** | Suppression des bordures d'un graphique Seaborn |
| **context** | Paramètre Seaborn contrôlant la taille des éléments selon l'usage |
| **style** | Paramètre Seaborn contrôlant l'apparence visuelle (fond, grille) |
| **colorblind** | Palette Seaborn accessible aux daltoniens — recommandée en publication |
| **AUC** | Area Under the Curve — mesure de performance d'un modèle de classification (0.5=aléatoire, 1.0=parfait) |
| **ROC** | Receiver Operating Characteristic — courbe TPR vs FPR |
| **confusion matrix** | Tableau récapitulatif des prédictions correctes et erreurs d'un classifieur |
| **learning curve** | Graphique score train/validation vs taille du dataset — diagnostique overfitting/underfitting |
| **PdfPages** | Classe Matplotlib pour générer un PDF multi-pages |
| **bbox_inches** | Paramètre de savefig — `"tight"` coupe les marges excédentaires |
