#python #ast #analyse-statique #module-standard #méta-programmation

## Rôle du module

`ast` est un module de la bibliothèque standard. Il permet de **parser du code Python en arbre syntaxique**, de le parcourir, de le transformer, puis éventuellement de revenir au code source. C'est la base des linters, formatters et outils d'analyse statique en Python.

Pour la théorie de l'AST en tant que structure de données, voir [[SD 11 — Arbre syntaxique abstrait (AST)]].

## Parser et visualiser

```python
import ast

source = "result = (a + b) * 2"
tree = ast.parse(source)          # → nœud Module racine

print(ast.dump(tree, indent=2))
# Module(
#   body=[
#     Assign(
#       targets=[Name(id='result', ctx=Store())],
#       value=BinOp(
#         left=BinOp(left=Name(id='a'), op=Add(), right=Name(id='b')),
#         op=Mult(),
#         right=Constant(value=2)))])
```

```python
ast.parse(source, mode="eval")    # expression seule → Expression(body=...)
ast.parse(source, mode="single")  # REPL interactif  → Interactive(body=...)
```

## Nœuds fondamentaux

| Nœud | Représente |
|---|---|
| `Module` | racine d'un fichier |
| `FunctionDef` / `AsyncFunctionDef` | définition de fonction |
| `ClassDef` | définition de classe |
| `Assign` / `AnnAssign` / `AugAssign` | affectation |
| `Return` / `Yield` | retour de valeur |
| `If` / `For` / `While` / `With` | structures de contrôle |
| `Import` / `ImportFrom` | imports |
| `Call` | appel de fonction |
| `BinOp` / `UnaryOp` / `BoolOp` | opérations |
| `Name` | référence à une variable |
| `Constant` | littéral (int, str, float…) |
| `Attribute` | accès `obj.attr` |
| `Subscript` | accès `obj[key]` |

Chaque nœud porte aussi `lineno` et `col_offset` (sauf les nœuds synthétiques).

## ast.walk() — parcours simple

Itère tous les nœuds en ordre pré-ordre, sans contrôle sur la descente.

```python
import ast

source = """
def foo(x):
    return x * 2

def bar():
    foo(42)
"""
tree = ast.parse(source)

# Lister tous les appels de fonctions
calls = [node for node in ast.walk(tree) if isinstance(node, ast.Call)]
for call in calls:
    if isinstance(call.func, ast.Name):
        print(call.func.id)   # foo
```

## NodeVisitor — parcours avec logique

Sous-classer `ast.NodeVisitor` pour déclencher du code sur des types de nœuds spécifiques.

```python
import ast

class FunctionLister(ast.NodeVisitor):
    def visit_FunctionDef(self, node: ast.FunctionDef) -> None:
        print(f"Fonction : {node.name} (ligne {node.lineno})")
        self.generic_visit(node)   # ← continuer la descente dans les enfants

source = """
def foo():
    def inner(): pass

class Bar:
    def method(self): pass
"""
FunctionLister().visit(ast.parse(source))
# Fonction : foo (ligne 2)
# Fonction : inner (ligne 3)
# Fonction : method (ligne 6)
```

> [!important] generic_visit est obligatoire
> Sans `self.generic_visit(node)` dans une méthode `visit_*`, la descente s'arrête : les nœuds enfants ne sont **pas** visités.

## NodeTransformer — modifier l'arbre

Même mécanique que `NodeVisitor`, mais chaque `visit_*` **retourne un nœud** (modifié, remplacé, ou `None` pour supprimer).

```python
import ast

class DoubleConstants(ast.NodeTransformer):
    def visit_Constant(self, node: ast.Constant) -> ast.Constant:
        if isinstance(node.value, (int, float)):
            return ast.Constant(value=node.value * 2)
        return node

source = "x = 3 + 1"
tree = ast.parse(source)
new_tree = DoubleConstants().visit(tree)
ast.fix_missing_locations(new_tree)   # ← remplir lineno/col manquants

print(ast.unparse(new_tree))   # x = 6 + 2
```

## ast.unparse() — retour au code source

Disponible depuis Python 3.9. Génère une représentation textuelle normalisée (pas identique à la source originale).

```python
import ast

tree = ast.parse("x  =  1+  2")
print(ast.unparse(tree))   # x = 1 + 2
```

## Compiler et exécuter un AST modifié

```python
import ast

source = "x = 1 + 2"
tree = ast.parse(source)
# ... transformer l'arbre ...
ast.fix_missing_locations(tree)
code = compile(tree, filename="<string>", mode="exec")
exec(code)
```

## Cas d'usage typiques

| Usage | Outil |
|---|---|
| Compter les appels à une fonction | `NodeVisitor` + `visit_Call` |
| Interdire certaines constructions | `NodeVisitor` + `visit_Import` |
| Injecter du code de logging | `NodeTransformer` + `visit_FunctionDef` |
| Renommer une variable partout | `NodeTransformer` + `visit_Name` |
| Vérifier la complexité cyclomatique | `NodeVisitor` + comptage `If`/`For`/`While` |
| Générer du code Python | construire l'AST à la main + `ast.unparse` |
