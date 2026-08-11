#ia #rag #chunking #intermédiaire

## L'étape qui pèse le plus sur la qualité d'un RAG

Le RAG cherche par similarité (voir [[RAG 02 — Embeddings et Vector Databases]]) : il compare le vecteur d'une question aux vecteurs des chunks indexés. Un vecteur résume tout le texte qu'il représente en un point unique — un chunk trop gros mélange plusieurs idées en un vecteur flou qui ne ressemble fortement à aucune question précise, un chunk trop petit perd son contexte (« il faut redémarrer le service » ne dit pas lequel).

> [!warning] Un mauvais chunking ne se rattrape pas en aval
> Cette étape n'a rien d'intelligent — c'est du découpage de chaîne de caractères — mais elle décide de ce que la recherche pourra, ou non, retrouver. Aucun réglage du LLM ou du prompt ne compense un corpus mal découpé.

## Taille fixe : robuste, aveugle au contenu

Une fenêtre glissante d'un nombre fixe de mots, qui avance dans le texte avec un recouvrement entre chunks voisins.

```python
def chunk_taille_fixe(texte, taille=60, recouvrement=12):
    """Découpe par fenêtre fixe de N mots, avec recouvrement entre voisins."""
    if recouvrement >= taille:
        raise ValueError("le recouvrement doit être inférieur à la taille")

    mots = texte.split()
    pas = taille - recouvrement
    chunks, debut = [], 0

    while debut < len(mots):
        chunks.append(" ".join(mots[debut:debut + taille]))
        if debut + taille >= len(mots):
            break
        debut += pas

    return chunks
```

Avantage : robuste, marche sur n'importe quel texte, structuré ou non. Défaut : coupe à l'aveugle, parfois en plein milieu d'une phrase — c'est le rôle du recouvrement d'en limiter les dégâts.

### Le rôle du recouvrement

Sans recouvrement, une idée à cheval sur deux chunks est perdue : sa première moitié finit un chunk, sa seconde ouvre le suivant, et aucun des deux vecteurs ne la capte entièrement. Le recouvrement (*overlap*) fait réapparaître les derniers mots d'un chunk au début du suivant — une idée coupée par la frontière se retrouve intacte dans au moins un chunk.

```python
chunks = chunk_taille_fixe(texte, taille=25, recouvrement=5)
# Les 5 derniers mots du chunk 1 sont aussi les 5 premiers du chunk 2.
```

> [!tip] 10 à 20 % de la taille du chunk
> Le recouvrement a un coût — du texte dupliqué, donc plus de chunks — mais il est presque toujours rentable. Une seule contrainte stricte : il doit rester inférieur à la taille du chunk, sinon le découpage n'avance plus (voir la validation `ValueError` dans le code ci-dessus).

## Par phrases : ne jamais couper une phrase

Regroupe des phrases entières sans jamais en casser une, jusqu'à un budget de mots.

```python
def chunk_par_phrases(texte, max_mots=60):
    """Regroupe des phrases entières sans dépasser un budget de mots."""
    phrases = segmenter_phrases(texte)  # segmentation en phrases, en amont
    chunks, courant, compte = [], [], 0

    for phrase in phrases:
        n = len(phrase.split())
        if courant and compte + n > max_mots:
            chunks.append(" ".join(courant))
            courant, compte = [], 0
        courant.append(phrase)
        compte += n

    if courant:
        chunks.append(" ".join(courant))

    return chunks
```

Un chunk se ferme dès que la phrase suivante ferait dépasser le budget — chaque chunk est donc toujours une suite de phrases complètes, toujours lisible. Contrepartie : une phrase très longue peut, à elle seule, dépasser le budget. C'est rare et acceptable — mieux vaut une phrase entière trop longue qu'une phrase tronquée.

## Par sections : suivre la structure déjà écrite

Quand le document est déjà structuré (Markdown avec des titres), le meilleur découpage est celui que l'auteur a déjà fait — une section, un chunk.

```python
def chunk_par_sections(markdown):
    """Découpe un Markdown sur ses titres : un chunk par section."""
    sections, titre, corps = [], "(préambule)", []

    for ligne in markdown.splitlines():
        if ligne.lstrip().startswith("#"):
            if any(l.strip() for l in corps):
                sections.append({"titre": titre, "texte": "\n".join(corps).strip()})
            titre, corps = ligne.lstrip("#").strip(), []
        else:
            corps.append(ligne)

    if any(l.strip() for l in corps):
        sections.append({"titre": titre, "texte": "\n".join(corps).strip()})

    return sections
```

> [!tip] Préfixer le titre au chunk — une amélioration gratuite
> Indexer le texte d'une section avec son titre en tête (« Volume nommé : créé explicitement, il est réutilisable ») enrichit le vecteur d'un contexte que le corps seul n'a pas. Un ajout quasi gratuit à la stratégie par sections.

C'est la stratégie la plus qualitative quand elle s'applique — elle suit le sens du document. Sa limite : elle exige un document structuré. Sur de la prose continue sans titres, elle n'a rien sur quoi s'appuyer.

## Choisir selon le corpus

| Corpus | Stratégie adaptée | Pourquoi |
|--------|----------------------|----------|
| Markdown, HTML structuré, code | Par sections | Suit le découpage voulu par l'auteur |
| Prose continue, articles | Par phrases | Respecte l'unité de sens minimale |
| Texte hétérogène, sans structure | Taille fixe + recouvrement | Robuste, marche partout |

> [!tip] La règle de décision
> Le document a une structure exploitable ? Découper par sections. Sinon, c'est de la prose ? Découper par phrases. Dans le doute, ou face à un corpus mélangé, la taille fixe avec recouvrement reste le repli fiable — beaucoup de pipelines combinent les trois selon le type de document rencontré.

Le chunking sémantique (voir [[RAG 02 — Embeddings et Vector Databases]]) va plus loin en coupant là où le sens change réellement, mesuré par similarité entre phrases voisines — plus fin, mais plus coûteux (un modèle d'embedding est nécessaire dès le découpage). Une optimisation à garder pour quand ces trois stratégies de base montrent leurs limites.

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| La recherche remonte des pavés | Chunks trop gros | Réduire la taille, viser une idée par chunk |
| Chunks incompréhensibles isolés | Chunks trop petits, contexte perdu | Augmenter la taille, préfixer le titre |
| Idées coupées aux frontières | Pas de recouvrement | Ajouter 10-20 % de recouvrement |
| `ValueError` sur le recouvrement | Recouvrement ≥ taille | Le rendre strictement inférieur à la taille |
| Chunking par sections vide | Document sans titres | Basculer sur phrases ou taille fixe |

## Pour aller plus loin

Une fois les chunks produits, l'étape suivante est leur transformation en vecteurs — voir [[RAG 02 — Embeddings et Vector Databases]]. Pour la construction complète du pipeline d'indexation, voir [[RAG 03 — Naive RAG]].

Sources : [Chunking pour le RAG — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/rag-chunking/)
