#ia #transformers #architecture #concept #fondamentaux

## Le chaînon manquant avant d'utiliser la librairie

Les fiches précédentes de ce module (voir [[TR 02 — Qu'est-ce que Hugging Face Transformers]]) montrent comment **charger et utiliser** un modèle avec `AutoModel`/`AutoTokenizer`. Cette fiche explique ce que ces quelques lignes de code chargent réellement : l'architecture Transformer elle-même, et son mécanisme central, l'attention.

## Avant 2017 : des approches qui traitaient le texte séquentiellement

| Époque | Approche | Limite |
|--------|----------|--------|
| 1950-1990 | Modèles statistiques (n-grammes) | Prédisent le mot suivant à partir des 2-3 mots précédents seulement, aucune compréhension du sens |
| Années 2000 | RNN, puis LSTM | Traitent le texte mot par mot dans l'ordre, gardent un peu de contexte, mais restent lents et peinent sur les dépendances longues |
| 2017 | **Transformer** (Google, *Attention Is All You Need*) | Traite tous les mots d'une phrase **en parallèle**, via le mécanisme d'attention |

> [!info] Pourquoi « Transformer »
> Le nom vient de sa capacité à transformer une séquence d'entrée en séquence de sortie en une seule passe, plutôt que mot par mot comme un RNN — cette parallélisation est ce qui a permis d'entraîner des modèles bien plus grands, à volume de calcul équivalent.

## La structure en couches

```
Texte brut
    ↓
Couche d'embedding        → transforme chaque token en vecteur numérique
    ↓
Bloc Transformer #1  ┐
Bloc Transformer #2  │  → dizaines de blocs empilés, chacun affine
Bloc Transformer #N  ┘     la représentation via l'attention
    ↓
Couche de sortie          → probabilité de chaque token du vocabulaire pour continuer
```

Chaque bloc Transformer analyse le texte pour en comprendre les relations internes — par exemple, dans « Le chien aboie parce qu'il a faim », comprendre que « il » désigne « chien ». C'est le rôle de l'attention.

## Self-attention : Query, Key, Value

Pour chaque token, le mécanisme d'attention calcule trois vecteurs dérivés de son embedding :

| Vecteur | Rôle |
|---------|------|
| **Query** (requête) | Ce que ce token « cherche » chez les autres tokens |
| **Key** (clé) | Ce que ce token « propose » comme information aux autres |
| **Value** (valeur) | Le contenu réellement transmis si ce token est jugé pertinent |

Le score d'attention entre deux tokens se calcule en comparant la Query de l'un à la Key de l'autre (un produit scalaire) — plus le score est élevé, plus la Value de ce token pèse dans la nouvelle représentation calculée. Chaque token peut ainsi « regarder » n'importe quel autre token de la séquence, quelle que soit sa distance, en une seule opération — exactement ce qu'un RNN ne pouvait faire qu'en propageant l'information pas à pas.

> [!tip] L'analogie d'une recherche documentaire
> Query = la question posée. Key = les mots-clés indexant chaque document. Value = le contenu du document. Le score Query·Key détermine quels documents (tokens) sont les plus pertinents pour répondre, et leur Value est ce qui est effectivement récupéré.

## Multi-head attention : plusieurs points de vue en parallèle

Un seul calcul d'attention ne capture qu'un type de relation. Les Transformers en font tourner **plusieurs en parallèle** (les « têtes »), chacune apprenant à se spécialiser :

```
Tête 1 → relations grammaticales (sujet/verbe)
Tête 2 → références pronominales (« il », « ses », « celui-ci »)
Tête 3 → associations sémantiques (sens, thème)
...
```

Les résultats des différentes têtes sont ensuite combinés, donnant une représentation qui superpose plusieurs types de relations captées simultanément.

> [!info] Un lien direct avec LoRA et Flash Attention
> Les adaptateurs LoRA (voir [[TR 11 — PEFT et LoRA]]) ciblent typiquement les matrices de projection Query/Key/Value plutôt que l'ensemble du modèle — c'est précisément parce que ce sont ces matrices qui concentrent l'essentiel de ce qu'un fine-tuning léger doit ajuster. Flash Attention (voir [[TR 14 — Optimisation (Flash Attention, torch.compile)]]) optimise le calcul de ce même mécanisme, sans en changer le résultat.

## Pourquoi l'ordre des mots doit être ajouté séparément

Le calcul d'attention lui-même ne « voit » pas l'ordre des tokens — comparer des Query/Key/Value ne change pas si on permute la séquence. Pour distinguer « Paul appelle Marie » de « Marie appelle Paul », des **tokens de position** sont ajoutés aux embeddings avant le premier bloc Transformer, encodant la position de chaque token dans la séquence.

## Ce que ça implique pour la fenêtre de contexte

Comparer chaque token à tous les autres a un coût qui augmente avec le carré de la longueur de la séquence — c'est directement pourquoi la fenêtre de contexte (voir [[TR 05 — Tokenisation en détail]] pour la tokenisation elle-même) reste une ressource limitée et coûteuse à agrandir, et pourquoi des techniques comme Flash Attention ou des architectures alternatives (Mamba) cherchent à réduire ce coût.

## Pour aller plus loin

Une fois le texte transformé en tokens puis en représentations attentionnelles, la génération proprement dite — comment le modèle choisit effectivement le token suivant — est couverte dans [[TR 06 — Génération de texte]].

Sources : [Anatomie d'un LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/llm/)
