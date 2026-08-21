#dev #regex #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Regex / Regexp** | Séquence de caractères définissant un motif de recherche pour trouver, valider ou manipuler du texte. |
| **Littéral** | Caractère qui correspond à lui-même dans le motif (ex. `abc` correspond à "abc"). |
| **Métacaractère** | Caractère spécial ayant une signification particulière dans une regex (`.`, `^`, `$`, `*`, `+`, `?`...) plutôt que d'être interprété littéralement. |
| **Classe de caractères** | Ensemble de caractères possibles à une position donnée, délimité par `[]` (ex. `[a-z]`). |
| **Classe prédéfinie** | Raccourci pour une classe courante : `\d` (chiffre), `\w` (alphanumérique), `\s` (espace blanc), et leurs négations `\D`, `\W`, `\S`. |
| **Quantificateur** | Symbole indiquant le nombre d'occurrences requises de l'élément précédent (`*`, `+`, `?`, `{n,m}`). |
| **Groupe capturant** | Sous-partie du motif entre parenthèses `()`, dont la correspondance est mémorisée et accessible séparément. |
| **Groupe non capturant** | Groupement `(?:...)` qui structure le motif (ex. pour l'alternance) sans mémoriser la sous-chaîne correspondante. |
| **Alternance** | Opérateur `\|` à l'intérieur d'un groupe, correspondant à l'une ou l'autre des options (`(a\|b)`). |
| **Lookahead** | Assertion qui vérifie ce qui suit une position sans consommer de caractères — positif `(?=...)`, négatif `(?!...)`. |
| **Lookbehind** | Assertion qui vérifie ce qui précède une position sans consommer de caractères — positif `(?<=...)`, négatif `(?<!...)`. |
| **BRE (Basic Regular Expression)** | Syntaxe regex basique POSIX, utilisée par défaut par `grep` et `sed` — `{}`, `+`, `?`, `\|` doivent être échappés pour agir comme métacaractères. |
| **ERE (Extended Regular Expression)** | Syntaxe regex étendue POSIX (activée avec `grep -E` / `egrep`, `sed -E`), plus proche de la syntaxe courante des autres langages. |
| **PCRE (Perl Compatible Regular Expressions)** | Implémentation de regex héritée de Perl, devenue la référence de fait dans la plupart des langages modernes (Python, JavaScript, Java...). |
