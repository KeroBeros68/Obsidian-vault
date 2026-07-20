#python #ast #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **AST (Abstract Syntax Tree)** | Arbre syntaxique abstrait représentant la structure d'un code source, sans ses détails de formatage (espaces, commentaires). |
| **`ast.parse()`** | Fonction transformant une chaîne de code Python en arbre AST, avec un mode (`exec` par défaut, `eval`, `single`). |
| **`ast.dump()`** | Fonction affichant la représentation textuelle complète d'un arbre AST, utile pour l'inspection et le débogage. |
| **Nœud (node)** | Élément de l'arbre AST représentant une construction du langage (`Module`, `FunctionDef`, `Call`, `Name`, `Constant`...). |
| **`lineno` / `col_offset`** | Attributs portés par la plupart des nœuds AST, indiquant leur position (ligne, colonne) dans le code source d'origine. |
| **`ast.walk()`** | Fonction parcourant tous les nœuds d'un arbre en ordre pré-ordre, sans possibilité de contrôler la descente. |
| **`ast.NodeVisitor`** | Classe à sous-classer pour parcourir un AST avec une logique personnalisée par type de nœud (`visit_FunctionDef`, `visit_Call`...), sans le modifier. |
| **`generic_visit()`** | Méthode à appeler explicitement dans un `visit_*` de `NodeVisitor`/`NodeTransformer` pour continuer la descente dans les nœuds enfants — sans elle, la descente s'arrête. |
| **`ast.NodeTransformer`** | Classe similaire à `NodeVisitor`, mais dont chaque méthode `visit_*` retourne un nœud (modifié, remplacé, ou `None` pour le supprimer), permettant de transformer l'arbre. |
| **`ast.fix_missing_locations()`** | Fonction complétant les attributs `lineno`/`col_offset` manquants sur les nœuds créés ou modifiés, requise avant de compiler un arbre transformé. |
| **`ast.unparse()`** | Fonction (Python 3.9+) régénérant une représentation textuelle normalisée du code à partir d'un arbre AST — pas nécessairement identique au texte source d'origine. |
| **`compile()`** | Fonction native transformant un arbre AST (ou du code source) en objet code exécutable, utilisable avec `exec()`. |
