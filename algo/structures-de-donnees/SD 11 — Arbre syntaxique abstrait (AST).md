#algorithmique #structures-de-données #ast #arbre #compilateur #parsing

## Définition

Un **AST** (Abstract Syntax Tree) est un arbre dont chaque nœud représente une construction du langage (instruction, expression, opérateur…). Il capture la **structure grammaticale** du code source, sans les détails syntaxiques superficiels (parenthèses, virgules, espaces).

```
Source : result = (a + b) * 2

         Assign
        /      \
     Name     BinOp (*)
   "result"  /       \
          BinOp (+)  Constant
          /     \       2
        Name   Name
         "a"   "b"
```

## Différence avec l'ABR

| Critère | ABR (BST) | AST |
|---|---|---|
| Propriété d'ordre | gauche < racine < droite | **aucune** — hiérarchie grammaticale |
| Type des nœuds | homogène (valeur comparable) | **hétérogène** (Assign, BinOp, Name…) |
| Construction | insertion de valeurs | **analyse syntaxique** (parsing) |
| Usage | recherche, tri | compilation, analyse, transformation |

## Du code source à l'AST

```
Source (texte)
    ↓ Lexer / tokenizer
Tokens  [NAME:"result", OP:"=", NAME:"a", OP:"+", ...]
    ↓ Parser (grammaire)
AST (arbre typé)
    ↓ Analyse sémantique / génération de code / ...
```

## Structure d'un nœud générique

```
Node {
    type     : string          -- Assign, BinOp, FunctionDef...
    children : Node[]          -- sous-nœuds (opérandes, corps, args...)
    metadata : any             -- ligne, colonne, type inféré...
}
```

## Catégories de nœuds courants

| Catégorie | Exemples |
|---|---|
| **Module / Programme** | racine de l'arbre complet |
| **Déclarations** | FunctionDef, ClassDef, Import |
| **Instructions** | Assign, Return, If, For, While |
| **Expressions** | BinOp, Call, Subscript, IfExp |
| **Atomes** | Name (variable), Constant (littéral) |
| **Opérateurs** | Add, Mult, Eq, And… (nœuds terminaux) |

## Parcours — patron Visiteur

Le parcours d'un AST suit le **patron Visiteur** : on sépare l'algorithme de parcours de la structure de l'arbre.

```
Visiteur
  visit(node):
    appeler visit_<TypeDuNœud>(node)
    parcourir récursivement les enfants

  visit_FunctionDef(node) → logique spécifique aux fonctions
  visit_Assign(node)      → logique spécifique aux affectations
  ...
```

**Parcours standard** : pré-ordre (racine → enfants), car il faut traiter le parent avant ses sous-expressions.

## Patron Transformateur

Variante du visiteur qui **retourne un nœud** : permet de réécrire l'arbre (optimisation, transpilation, instrumentation).

```
Transformateur
  visit_BinOp(node):
    si opération constante → retourner Constant(eval(node))
    sinon                  → retourner node inchangé (ou modifié)
```

## Algorithme de parcours (pseudo-code)

```
def walk(node):
    yield node
    for child in node.children:
        yield from walk(child)         # pré-ordre, DFS

def visit(node, visitor):
    handler = visitor["visit_" + node.type]
    if handler: handler(node)
    for child in node.children:
        visit(child, visitor)
```

## Applications

| Domaine | Usage |
|---|---|
| Compilateurs | génération de bytecode / code machine |
| Interpréteurs | évaluation directe de l'AST |
| Linters (pylint, ESLint) | détecter des patterns problématiques |
| Formatters (black, prettier) | réimprimer le code depuis l'AST |
| Refactoring automatique | renommer, extraire, inliner |
| Analyse statique | détection de bugs, typage |
| Transpileurs | CoffeeScript → JS, TypeScript → JS |
| Méta-programmation | génération de code à la compilation |

## Complexités

| Opération | Complexité |
|---|---|
| Parsing (construction) | O(n) — n = nombre de tokens |
| Parcours complet | O(n) — n = nombre de nœuds |
| Recherche d'un pattern | O(n) |
| Transformation (copie) | O(n) |

> [!tip] Lien avec les autres arbres
> L'AST peut contenir des sous-arbres ressemblant à un ABR (expressions arithmétiques triées par précédence) ou à un Trie (résolution de noms dans les scopes imbriqués). En Python, le module standard `ast` expose directement cet arbre — voir [[AST Python 01 — Le module ast]].
