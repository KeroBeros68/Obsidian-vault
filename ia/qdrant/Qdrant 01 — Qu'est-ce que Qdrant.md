#ia #qdrant #vectordb #fondamentaux

## Une base vectorielle serveur, pas embarquée

FAISS et Chroma (voir [[Chroma — Index des fiches]]) vivent dans le processus Python qui les utilise — parfait pour prototyper, rien à déployer, mais limité dès qu'on industrialise. **Qdrant** répond à un besoin différent : c'est une base vectorielle **serveur**, écrite en Rust, qui s'exécute comme un service indépendant auquel le code se connecte par le réseau.

## Pourquoi ça change tout en production

Un RAG en production a des besoins qu'une base embarquée ne couvre pas : plusieurs applications (une API, un chatbot, un job d'indexation) doivent interroger le **même index** en même temps, cet index doit **survivre** au redémarrage de n'importe lequel de ces clients, et il doit tenir la charge de requêtes concurrentes.

> [!info] Le filtrage par payload, l'atout distinctif
> Au-delà du modèle client-serveur, Qdrant ajoute un filtrage par métadonnées (*payload*) de premier ordre — la capacité qui rend possible le **multi-tenant** : plusieurs équipes ou clients partagent une seule collection, mais chaque recherche reste cloisonnée à un périmètre précis. Voir [[Qdrant 04 — Rechercher & filtrer par payload]].

## Où se situe Qdrant dans une progression de projet

| Étape du projet | Base vectorielle |
|---------------------|----------------------|
| Prototype jetable | FAISS |
| Développement, métadonnées | Chroma |
| Production, service partagé, filtrage | **Qdrant** |

> [!tip] Le principe ne change jamais, seule la robustesse évolue
> Vectoriser, indexer, rechercher — ce triptyque reste identique de FAISS à Qdrant en passant par Chroma (voir [[RAG 02 — Embeddings et Vector Databases]]). Passer d'une base à l'autre ne remet pas en cause l'architecture d'un RAG, seulement la robustesse du stockage et les capacités de filtrage disponibles.

## Pour aller plus loin

Lancer une première instance Qdrant et s'y connecter en Python est couvert dans [[Qdrant 02 — Installation & lancement]].

Sources : [Qdrant — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/qdrant/)
