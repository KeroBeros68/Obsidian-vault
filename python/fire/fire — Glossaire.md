#python #fire #cli #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **composant** | Objet Python exposé par Fire (fonction, classe, instance, dict, module). |
| **fire.Fire(component)** | Point d'entrée unique. Inspecte `component` et génère le CLI correspondant. |
| **flag positionnel** | Argument passé sans nom, dans l'ordre des paramètres. Ex : `python s.py Kevin 42`. |
| **flag nommé** | Argument passé avec `--nom=valeur` ou `--nom valeur`. Ordre libre. |
| **flag booléen** | `--flag` → `True`, `--noflag` → `False`. Spécifique à Fire (pas `--no-flag`). |
| **fire flag** | Option globale de Fire placée après `--`. Ex : `-- --help`, `-- --interactive`. |
| **séparateur `--`** | Double tiret : sépare les arguments du composant des fire flags. |
| **séparateur de chaîne** | Tiret simple `-` par défaut : sépare les appels chainés. Modifiable via `-- --separator=X`. |
| **chaînage (chaining)** | Appel successif de méthodes sur le résultat d'une commande. Ex : `cmd1 arg - cmd2 arg2`. |
| **composant imbriqué** | Attribut d'une classe Fire qui est lui-même un objet exposable (classe, dict, instance). |
| **coercition de type** | Conversion automatique de la chaîne CLI vers le type Python annoté. |
| **`-- --interactive`** | Fire flag ouvrant un REPL Python/IPython avec le résultat dans la variable `result`. |
| **`-- --completion`** | Fire flag générant le script d'autocomplétion pour bash, zsh ou fish. |
| **`@fire.decorators.SetParseFn`** | Décorateur Fire pour personnaliser la fonction de parsing d'un argument spécifique. |
| **`-- --trace`** | Fire flag affichant la trace interne d'exécution — utile pour déboguer le comportement de Fire. |
