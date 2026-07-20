# Template — Nouvelle catégorie

Ce fichier documente les conventions du vault. Copier-coller les blocs ci-dessous pour créer une nouvelle catégorie, quel que soit le domaine.

---

## Arborescence

Le vault est organisé par **domaine** puis par **module**. Le niveau intermédiaire est optionnel si le domaine est simple ; il devient une **sous-catégorie spécialisée** quand plusieurs modules du domaine couvrent la même famille de technologies (ex. conteneurs, orchestration).

```
[domaine]/
└── [module]/
    ├── [Module] — Index des fiches.md
    ├── [Module] — Glossaire.md
    ├── [Module] — Pièges classiques.md   ← optionnel selon le domaine
    ├── [Module] 01 — [Titre].md
    ├── [Module] 02 — [Titre].md
    └── ...

[domaine]/[sous-catégorie]/[module]/   ← variante avec niveau intermédiaire
```

**Exemples de domaines :**

```
python/numpy/
python/fastapi/
cpp/stl/
cpp/memory/
rust/ownership/
js/promises/
java/collections/
csharp/linq/
zig/comptime/
math/algebre-lineaire/
math/probabilites/
algo/graphes/
algo/tri/
devops/containers/docker/       ← sous-catégorie "containers" (regrouperait aussi Kubernetes, Podman...)
devops/web/nginx/                ← sous-catégorie "web" (regrouperait aussi Apache, Caddy...)
devops/secrets/
bdd/sql/                          ← domaine "bdd" (langage de requête, transversal aux moteurs)
bdd/relationnelles/mysql/        ← sous-catégorie "relationnelles" (regrouperait aussi PostgreSQL, SQLite...)
ia/transformers/
ia/rag/
```

Ajouter chaque nouveau module dans [[Home]].

---

## Index des fiches

```markdown
#[domaine] #[module] #[tag-optionnel]

## Fiches disponibles

### Fondamentaux

- [[Module 01 — Titre]]
- [[Module 02 — Titre]]
- [[Module 03 — Titre]]

### Intermédiaire

- [[Module 04 — Titre]]
- [[Module 05 — Titre]]

### Avancé

- [[Module 06 — Titre]]

### Référence

- [[Module — Glossaire]]
- [[Module — Pièges classiques]]

## Prérequis & suite

- [[Autre module — Index des fiches]] ← prérequis
- [[Autre module — Index des fiches]] ← suite logique
```

---

## Fiche numérotée — variante code

Pour les modules orientés langage (Python, C++, Rust, JS, etc.).

```markdown
#[domaine] #[module] #[sujet]

## [Concept principal]

[Intro courte si nécessaire — une phrase max.]

```[langage]
// exemple minimal
code_ici();
```

## [Cas courants]

| Situation | Syntaxe / Approche |
|-----------|-------------------|
| cas 1     | `code`            |
| cas 2     | `code`            |

## [Comportement ou subtilité]

```[langage]
bon_usage();      // ✅ comportement attendu
cas_limite();     // ⚠️ attention
mauvais_usage();  // ❌ erreur ou UB
```

> [!tip] Titre
> Règle mémo ou pourquoi c'est ainsi.

> [!warning] Titre
> Ce qui surprend ou ce qu'on oublie facilement.
```

**Identifiants de langage courants :**
`python` `cpp` `c` `rust` `zig` `java` `csharp` `js` `ts` `go` `bash` `sql` `dockerfile` `yaml`

---

## Fiche numérotée — variante théorique

Pour les modules sans code dominant (math, algo, IA, concepts DevOps, etc.).

```markdown
#[domaine] #[module] #[sujet]

## [Concept]

[Définition ou énoncé clair.]

## [Propriétés / Règles]

- Propriété 1 : ...
- Propriété 2 : ...

## [Illustration]

[Exemple concret, diagramme ASCII, formule, pseudo-code, ou bloc de code si pertinent.]

```
entrée → [étape 1] → [étape 2] → sortie
```

## [Cas particuliers]

> [!warning] Titre
> Ce qui ne s'applique pas dans tel contexte.

> [!tip] Titre
> Intuition ou analogie utile.
```

---

## Glossaire

```markdown
#[domaine] #[module] #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **terme1** | Définition précise. Ex : `code` ou formule. |
| **terme2** | Définition précise. |
| **terme3** | Définition précise. |
```

---

## Pièges classiques

Pertinent pour les langages et les domaines techniques. Optionnel pour les domaines purement théoriques.

```markdown
#[domaine] #[module] #pièges #erreurs #debugging

## 🪤 Piège 1 — [Titre court]

```[langage]
mauvaise_approche();  // ❌ pourquoi ça pose problème
bonne_approche();     // ✅
```

> [!warning] Titre
> Ce qui se passe concrètement.

---

## 🪤 Piège 2 — [Titre court]

[Description du piège, avec ou sans code selon le domaine.]

> [!tip] Mémo
> Règle simple à retenir.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Description courte | Solution courte |
| Description courte | Solution courte |
```

---

## Conventions

### Nommage des fichiers

- Séparateur : ` — ` (espace + tiret long + espace)
- Numérotation sur 2 chiffres : `01`, `02` ... `12`
- Fichiers spéciaux sans numéro : `Module — Index des fiches`, `Module — Glossaire`, `Module — Pièges classiques`

### Tags (première ligne du fichier, pas de frontmatter YAML)

- **Tag 1** = domaine : `#python` `#cpp` `#rust` `#js` `#java` `#csharp` `#zig` `#math` `#algo` `#devops` `#ia`
- **Tag 2** = module : `#numpy` `#stl` `#ownership` `#graphes` `#docker` `#transformers`
- **Tags suivants** = sujet de la fiche : `#bases` `#avancé` `#référence` `#pièges` `#mémoire` `#async`

### Callouts

| Callout | Usage |
|---------|-------|
| `[!tip]` | Astuce, règle mémo, pattern recommandé, intuition |
| `[!warning]` | Piège, comportement contre-intuitif, erreur silencieuse, UB |
| `[!info]` | Information contextuelle neutre, prérequis, version |
| `[!example]` | Exemple étendu qui alourdirait le flux principal |

### Code

- Toujours spécifier le langage après les triples backticks
- Annoter ✅ / ❌ / ⚠️ dans les commentaires inline
- Utiliser la syntaxe de commentaire du langage cible : `//` `#` `--` `%%` `;;`
- Exemples minimaux — pas de setup inutile
