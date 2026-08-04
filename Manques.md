#manques #todo #prérequis

# Manques du vault — par priorité

Modules absents référencés dans des fiches existantes ou représentant une continuation naturelle des parcours couverts. Classés par impact sur la navigation et la compréhension.

**État du vault au 2026-07-02 :** 357 fiches numérotées — Python (118), IA (111), Bash (9), C (38), Algo (39), SQL (12), Docker (8), Secrets (6), Probabilités (9), Math (7)

---

## Priorité 2 — Bloquants partiels

Ces modules créent des **liens morts actifs** dans des fiches existantes. Leur absence laisse des wikilinks cassés dans 2 index ou plus.

### FastAPI
- **Liens morts dans :** Typing — Index (`[[FastAPI — Index des fiches]]`) ; Pydantic — Index (`[[FastAPI — Index des fiches]]`) ; Docker — Index (`[[FastAPI — Index des fiches]]`)
- **Mentions via `[[Manques]]` dans :** asyncio — Index ; Déco — Index
- **Contenu attendu :** routing ASGI, path/query params, request body (Pydantic), réponses typées, middleware, dépendances (`Depends`), authentification JWT, `HTTPException`, background tasks, lifespan
- **Parcours :** Typing → Pydantic → asyncio → **FastAPI** → SQLAlchemy

### SQLAlchemy
- **Liens morts dans :** Pydantic — Index (`[[SQLAlchemy — Index des fiches]]`)
- **Mentions via `[[Manques]]` dans :** SQL — Index
- **Contenu attendu :** modèles ORM (`DeclarativeBase`), sessions (`Session`, `AsyncSession`), requêtes (`select`, `where`, `join`), relations (`relationship`), migrations Alembic, intégration async
- **Parcours :** Bases Python → OOP → **SQLAlchemy** → Pydantic 14 → FastAPI

---

## Priorité 3 — Suite logique

Ces modules ne créent pas de liens morts mais représentent des **continuations naturelles indispensables** des parcours couverts.

### ML / scikit-learn
- **Référencé dans :** Pandas 10 — Préparation ML (`train_test_split`, `StandardScaler`) ; Matplotlib & Seaborn 09 — Graphiques ML (`confusion_matrix`, `roc_curve`, `learning_curve`)
- **Contenu attendu :** pipeline ML, preprocessing (`LabelEncoder`, `OneHotEncoder`, `StandardScaler`), `sklearn.model_selection`, `sklearn.metrics`, validation croisée, modèles classiques (régression, forêts, SVM, KNN)
- **Parcours :** NumPy → Pandas → Algèbre linéaire → **ML/scikit-learn** → Deep Learning

### Deep Learning / PyTorch
- **Contexte :** grand vide dans le parcours IA — le vault couvre les fondamentaux LLM et le fine-tuning de modèles existants, mais pas la construction de réseaux de neurones. Pont nécessaire entre ML/scikit-learn et Fine-tuning FT 04 (LoRA/QLoRA)
- **Contenu attendu :** tenseurs PyTorch, autograd, `nn.Module`, couches (`Linear`, `Conv2d`, `Embedding`), boucle d'entraînement, optimiseurs, DataLoader, GPU (`cuda`/`mps`), transfer learning
- **Parcours :** ML/scikit-learn → **Deep Learning/PyTorch** → Fine-tuning FT 04-05

### Programmation dynamique
- **Contexte :** quatrième pilier algorithmique — Tri (13 fiches), Arbres (6 fiches) et Graphes (9 fiches) couverts, DP est le chaînon manquant pour compléter l'algorithmique classique
- **Contenu attendu :** mémoïsation top-down, DP bottom-up, sous-problèmes chevauchants, optimal substructure ; problèmes classiques : sac à dos (0/1 et fractionnaire), LCS, edit distance, coin change, plus long chemin dans un DAG
- **Parcours :** SD → Tri → Arbres → Graphes → **Programmation dynamique**

### Flux dans les réseaux
- **Lien mort dans :** Graphes — Index (`[[Manques]] ← Flux dans les réseaux (P3, non couvert)`)
- **Contenu attendu :** flot maximum, coupe minimale (théorème max-flow min-cut), Ford-Fulkerson, Edmonds-Karp, Dinic, matching bipartite, applications (transport, planification)
- **Parcours :** Graphes → SD → **Flux dans les réseaux**

### LlamaIndex
- **Contexte :** LangChain couvert (16 fiches dans `ia/langchain/`), LlamaIndex est le second framework RAG/agents majeur — plus orienté indexation de documents et requêtage de données structurées
- **Contenu attendu :** `VectorStoreIndex`, `QueryEngine`, node parsers, retrievers, agents LlamaIndex, `RouterQueryEngine`, comparaison architecturale avec LangChain
- **Parcours :** RAG → BM25S → LangChain → **LlamaIndex**

### Makefile & build C
- **Contexte :** Bases C 01 couvre `gcc` seul — tout projet C multi-fichier réel nécessite make ; absence bloquante pour Posix et Pthread (projets système)
- **Contenu attendu :** règles explicites/implicites, variables (`CC`, `CFLAGS`, `LDFLAGS`, `LIBS`), phony targets, compilation séparée (`.o`), librairies statiques (`.a`) et dynamiques (`.so`)
- **Parcours :** Bases C → **Makefile** → Posix → Pthread

### I/O fichiers en C
- **Contexte :** Bases 08 couvre `printf`/`scanf` (stdout/stdin) ; Posix 04 couvre les descripteurs bas niveau (`open`/`read`/`write`) — l'API `FILE*` standard C n'est couverte nulle part
- **Contenu attendu :** `fopen`/`fclose`, `fread`/`fwrite`, `fgets`/`fputs`, `fprintf`/`fscanf`, `fseek`/`ftell`/`rewind`, `feof`/`ferror`, `FILE*` vs `int fd` — quand utiliser l'un ou l'autre
- **Parcours :** Bases C → **I/O fichiers** → Posix 04

---

## Priorité 4 — Suites avancées

Modules référencés dans des index comme suites avancées, ou prérequis pour des domaines non encore entamés. Utiles mais non bloquants pour les parcours actuels.

### stdatomic C11
- **Lien mort dans :** Posix — Index (`[[stdatomic — Index des fiches]]`) ; Pthread — Index (suite logique après mutex/spinlocks)
- **Contenu attendu :** `_Atomic`, opérations atomiques (`atomic_fetch_add`, `atomic_compare_exchange_strong`), memory ordering (`seq_cst`, `relaxed`, `acquire/release`), CAS, lock-free data structures
- **Parcours :** Pthread → **stdatomic** → Kernel Linux

### Kernel Linux — Synchronisation
- **Lien mort dans :** Posix — Index (`[[Kernel Linux — Synchronisation]]` comme suite avancée)
- **Contenu attendu :** `futex`, `clone(2)`, `epoll`, `io_uring`, namespaces, cgroups, appels système bas niveau — niveau programmation kernel/système
- **Parcours :** Posix → Pthread → stdatomic → **Kernel Linux**

### Calcul différentiel
- **Contexte :** prérequis mathématique manquant pour comprendre le Deep Learning — backpropagation, gradient descent, optimisation
- **Contenu attendu :** dérivées, règle de la chaîne, gradient, gradient descent, dérivées partielles, jacobien, hessien, optimisation convexe
- **Parcours :** Algèbre linéaire → Probabilités → **Calcul différentiel** → Deep Learning

### Kubernetes
- **Contexte :** suite naturelle de Docker — DevOps — Home et Docker — Index le référencent comme étape suivante. Tout déploiement d'app en production à l'échelle passe par Kubernetes
- **Contenu attendu :** pods, services, deployments, ConfigMap/Secret, PersistentVolume, Ingress, Helm, kubectl, namespaces, HorizontalPodAutoscaler
- **Parcours :** Docker → **Kubernetes**

### CI/CD (Pipelines DevOps)
- **Contexte :** pont entre le développement local et la production — build automatisé, tests, push vers un registry, déploiement. Naturellement à cheval entre Docker et un futur module `devops/ci-cd/`
- **Contenu attendu :** GitHub Actions, GitLab CI, concepts de pipeline (build → test → push → deploy), cache de build en CI, déclencheurs (push, PR, tag), artefacts, secrets CI
- **Parcours :** Docker → Bash → **CI/CD** → Kubernetes

### Monitoring & Observabilité
- **Lien mort dans :** Cron — Index (suite logique, centralisation des logs au-delà de `journalctl`/MAILTO)
- **Contexte :** au-delà de `docker logs` et `docker stats`, la supervision réelle d'applications conteneurisées en production nécessite une stack dédiée — mentionné en creux dans LLMOps et Docker
- **Contenu attendu :** Prometheus + Grafana (métriques), ELK ou Loki (logs centralisés), alerting, dashboards, intégration dans Docker Compose et Kubernetes
- **Parcours :** Docker → LLMOps → **Monitoring** → Kubernetes

---

## Récapitulatif

| Module                         | Priorité | Liens morts / Débloque                                       |
| ------------------------------ | -------- | ------------------------------------------------------------ |
| FastAPI                        | 🟠 P2    | 3 liens morts (Typing, Pydantic, Docker) + 2 via [[Manques]] |
| SQLAlchemy                     | 🟠 P2    | 1 lien mort (Pydantic Index) + 1 via [[Manques]] (SQL)       |
| ML / scikit-learn              | 🟡 P3    | Pandas 10, Matplotlib 09, pont vers Deep Learning            |
| Deep Learning / PyTorch        | 🟡 P3    | pont ML → Fine-tuning, compréhension backprop                |
| Programmation dynamique        | 🟡 P3    | 4e pilier algo après Tri + Arbres + Graphes                  |
| Flux dans les réseaux          | 🟡 P3    | 1 lien mort (Graphes — Index), Ford-Fulkerson                |
| LlamaIndex                     | 🟡 P3    | 2e framework RAG/agents, complément LangChain                |
| Makefile & build C             | 🟡 P3    | projets C multi-fichiers, prérequis Posix réel               |
| I/O fichiers en C              | 🟡 P3    | gap Bases 08 → Posix 04, API `FILE*`                         |
| stdatomic C11                  | 🔵 P4    | 2 liens morts (Posix, Pthread), lock-free                    |
| Kernel Linux — Synchronisation | 🔵 P4    | 1 lien mort (Posix — Index), bas niveau système              |
| Calcul différentiel            | 🔵 P4    | prérequis backprop / Deep Learning                           |
| Kubernetes                     | 🔵 P4    | suite Docker, déploiement à l'échelle                        |
| CI/CD (Pipelines DevOps)       | 🔵 P4    | build/test/push/deploy automatisé, pont Docker → Kubernetes  |
| Monitoring & Observabilité     | 🔵 P4    | 1 lien mort (Cron — Index), Prometheus/Grafana/ELK, supervision conteneurs et LLM |

https://claude.ai/code/artifact/b6e14aa4-1139-48e1-9046-9ea03913d5f3?via=auto_preview