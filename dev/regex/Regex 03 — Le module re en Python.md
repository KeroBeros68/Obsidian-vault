#dev #regex #python

## Les fonctions principales

```python
import re
```

| Fonction | Comportement |
|---|---|
| `re.search(pattern, text)` | Cherche la 1ère correspondance n'importe où dans la chaîne — retourne un `Match` ou `None` |
| `re.match(pattern, text)` | Cherche uniquement au **début** de la chaîne |
| `re.findall(pattern, text)` | Retourne la liste de **toutes** les correspondances |
| `re.sub(pattern, repl, text)` | Remplace les correspondances par `repl` |
| `re.compile(pattern)` | Précompile la regex pour un usage répété (plus performant) |

```python
# re.search() : trouve la première occurrence n'importe où
pattern = r"\b\d{3}-\d{2}-\d{4}\b"
text = "Mon numéro de sécurité sociale est 123-45-6789."
match = re.search(pattern, text)
if match:
    print(f"Trouvé : {match.group()}")

# re.match() : ne cherche qu'au début de la chaîne
pattern = r"\d{3}-\d{2}-\d{4}"
text = "123-45-6789 est mon numéro."
match = re.match(pattern, text)   # ✅ correspond, le motif est en tête
match = re.match(pattern, "Mon numéro est 123-45-6789.")  # ❌ None, pas en tête

# re.findall() : toutes les occurrences
pattern = r"\b\w+@\w+\.\w+\b"
text = "Contactez-nous à a@ex.com ou b@ex.com."
emails = re.findall(pattern, text)   # ['a@ex.com', 'b@ex.com']

# re.sub() : rechercher-remplacer
pattern = r"\b\d{3}-\d{2}-\d{4}\b"
result = re.sub(pattern, "XXX-XX-XXXX", "Mon numéro est 123-45-6789.")

# re.compile() : précompiler pour réutilisation
compiled = re.compile(r"\b\d{3}-\d{2}-\d{4}\b")
matches = compiled.findall("Numéro 1 : 123-45-6789, Numéro 2 : 987-65-4321.")
```

> [!tip] `re.compile()` pour les motifs réutilisés
> Si le même motif est appliqué plusieurs fois (boucle, gros volume de texte), le compiler une fois avec `re.compile()` évite de le reparser à chaque appel.

## Groupes et captures

Comme en [[Regex 01 — Syntaxe et métacaractères|regex générique]], les parenthèses capturent des sous-parties du texte, accessibles via `match.group(n)`.

```python
pattern = r"(\d{3})-(\d{2})-(\d{4})"
text = "Numéro de sécurité sociale : 123-45-6789."
match = re.search(pattern, text)
if match:
    print(match.group(1))   # "123"
    print(match.group(2))   # "45"
    print(match.group(3))   # "6789"
```

## Les flags

Passés en dernier argument des fonctions `re`, ils modifient le comportement de la recherche.

| Flag | Effet |
|---|---|
| `re.IGNORECASE` / `re.I` | Ignore la casse |
| `re.MULTILINE` / `re.M` | `^` et `$` correspondent au début/fin de **chaque ligne**, pas seulement de la chaîne entière |
| `re.DOTALL` / `re.S` | `.` correspond aussi aux sauts de ligne |

```python
pattern = r"^bonjour"
text = "Bonjour tout le monde\nbonjour tout le monde"
matches = re.findall(pattern, text, re.IGNORECASE | re.MULTILINE)
# trouve "Bonjour" ET "bonjour", en début de chaque ligne grâce à MULTILINE
```

## Exercices

```python
# Valider une adresse IP
pattern = r"^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$"
match = re.match(pattern, "192.168.1.1")   # ✅ valide

# Extraire les hashtags d'un texte
text = "Suivez-nous sur #Python #regex #coding!"
hashtags = re.findall(r"#\w+", text)   # ['#Python', '#regex', '#coding']
```

## Prérequis & suite

- [[Regex — Index des fiches]] ← retour à l'index du module
- [[Regex 01 — Syntaxe et métacaractères]] ← prérequis : syntaxe des motifs utilisés ici
- [[Regex 02 — grep, sed et awk en Bash]] ← les mêmes motifs appliqués en ligne de commande
- [[Python — Home]] ← domaine Python plus large
