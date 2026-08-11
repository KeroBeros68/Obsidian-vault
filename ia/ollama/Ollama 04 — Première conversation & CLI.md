#ia #ollama #fondamentaux

## Session interactive

```bash
ollama run llama3.2
```

Après quelques secondes de chargement, une invite `>>>` attend une question :

```
>>> Bonjour ! Peux-tu te présenter ?
Bonjour ! Je suis un assistant IA basé sur le modèle Llama 3.2...
Je fonctionne entièrement sur votre ordinateur grâce à Ollama, ce qui
signifie que vos conversations restent privées et ne sont envoyées nulle part.
```

Pour quitter la session :

```
>>> /bye
```

## Questions ponctuelles, sans session interactive

```bash
ollama run llama3.2 "Quelle est la capitale de l'Australie ?"
```

La réponse s'affiche et la main revient immédiatement au terminal — pratique pour une question rapide ou pour intégrer Ollama dans un script shell.

## Utiliser plusieurs modèles

```bash
ollama run mistral "Écris un haïku sur l'automne"
ollama run codellama "Écris une regex pour valider une adresse email"
```

Chaque modèle installé (voir [[Ollama 03 — Télécharger et gérer des modèles]]) reste disponible indépendamment — changer de modèle revient simplement à changer le nom passé à `ollama run`.

## Formuler de meilleures questions

Les mêmes principes de formulation que pour n'importe quel LLM s'appliquent (voir [[IA 05 — L'art du prompt]]) : une question vague produit une réponse générique, une question contextualisée produit une réponse exploitable.

```
❌ "Parle-moi de Python"
✅ "Explique les différences entre les listes et les tuples en Python,
    avec des exemples de cas où utiliser l'un plutôt que l'autre"

❌ "Écris du code"
✅ "Écris une fonction Python qui prend une liste de nombres et retourne
    la moyenne, en gérant le cas où la liste est vide. Ajoute des
    commentaires explicatifs."
```

## Pour aller plus loin

Le mode ligne de commande suffit pour explorer un modèle, mais l'intégration dans une application passe par l'API REST (port `11434`) ou le SDK Python d'Ollama — la sécurisation de cette API avant toute exposition réseau est couverte dans [[Ollama 05 — Sécurité réseau (OLLAMA_HOST & port 11434)]].

Sources : [Installer et utiliser Ollama — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/ollama/)
