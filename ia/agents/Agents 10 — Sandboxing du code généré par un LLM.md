#ia #agents #sécurité #docker #avancé

## Le risque : un agent qui exécute du code

Un agent qui écrit du code est un agent qui **exécute** du code — du code produit par un modèle, donc faillible, et exploitable par une injection de prompt. Trois scénarios mènent au même désastre :

- **L'erreur du modèle** : un LLM imparfait génère une commande destructrice en croyant bien faire.
- **L'injection de prompt** : un agent qui lit une page web piégée se voit dicter du code hostile (voir [[MCP — Pièges classiques]] pour ce même mécanisme appliqué aux outils).
- **L'agent exposé** : une interface publique où un attaquant fabrique une entrée qui détourne l'exécution.

> [!warning] Le code lancé sans isolation hérite des droits du processus agent
> Lire vos fichiers, exfiltrer des secrets par le réseau, abuser d'API, attaquer le réseau interne — la parade n'est pas de faire confiance au modèle, c'est de partir du principe que **le code est hostile** et de l'exécuter dans un environnement où, hostile ou non, il ne peut rien atteindre d'important. C'est une application directe du principe du moindre privilège déjà vu dans [[Agents 03 — Les outils (Tools)]], poussée à l'exécution de code arbitraire plutôt qu'à un appel d'outil borné.

## Le spectre de l'isolation

Isoler n'est pas binaire : c'est un gradient, du plus léger au plus étanche, où plus l'isolation est forte, plus le coût de mise en place et la latence montent.

| Solution | Isolation | Coût de mise en place | Latence | Pour quel code |
|---|---|---|---|---|
| **Interpréteur restreint** | Faible, même processus | Nul | Négligeable | Code peu risqué, modèle de confiance |
| **Conteneur durci** | Bonne, noyau partagé | Faible (Docker) | Démarrage conteneur | Défaut raisonnable |
| **gVisor** | Forte, noyau applicatif | Moyen (runtime à installer) | Faible surcharge | Code peu fiable, multi-tenant |
| **microVM (Firecracker)** | Très forte, VM dédiée | Élevé (infra) | Démarrage VM (~ms) | Code non fiable, exposition publique |
| **Service géré (E2B, Daytona…)** | Très forte, déléguée | Nul (compte) | Réseau + démarrage | Pas d'infra à gérer, budget dédié |

L'**interpréteur restreint** (celui de smolagents, voir [[Manques]]) analyse le code avant de l'exécuter et bloque les imports dangereux — léger, sans dépendance, mais il s'exécute dans votre processus : une faille de l'interpréteur, et l'isolation tombe.

Le **conteneur durci** exécute le code dans un conteneur Docker dépouillé de tout privilège — accessible partout où Docker tourne, bonne isolation, mais il partage le noyau de l'hôte.

**gVisor** insère un noyau applicatif entre le conteneur et le vrai noyau : les appels système sont interceptés et traités par gVisor, jamais directement par l'hôte. La **microVM** (Firecracker) va plus loin : une vraie machine virtuelle avec son propre noyau, démarrée en quelques dizaines de millisecondes — la plus forte isolation, aussi ce que proposent les services gérés comme E2B ou Daytona.

> [!tip] La règle de décision
> Code plutôt fiable, modèle connu : un conteneur durci suffit. Code non fiable ou agent exposé au public : il faut gVisor ou une microVM. Quand on ne veut pas gérer l'infrastructure, un service géré déplace le problème, au prix d'une dépendance et d'une facture.

## Construire un bac à sable Docker durci

Le conteneur durci offre le meilleur rapport isolation/simplicité, et reste reproductible partout où Docker tourne. L'idée : exécuter le code dans un conteneur jetable, lancé avec un jeu d'options qui lui retirent tout pouvoir. Le mécanisme sous-jacent (capabilities Linux, utilisateur non-root, namespaces) est détaillé dans [[Docker 08 — Sécurité des conteneurs]] et [[Docker 11 — Sous le capot (namespaces, cgroups, seccomp)]] — ce qui suit est leur application ciblée à l'exécution de code généré par un LLM.

```python
IMAGE = (
    "python:3.12-slim@sha256:"
    "9d3abd9fc11d06998ccdbdd93b4dd49b5ad7d67fcbbc11c016eb0eb2c2194891"
)

DURCISSEMENT = [
    "--network", "none",              # aucune connexion réseau
    "--cap-drop", "ALL",              # aucune capability Linux
    "--security-opt", "no-new-privileges",  # pas d'escalade de privilèges
    "--read-only",                    # racine en lecture seule
    "--tmpfs", "/tmp:size=16m",       # seul /tmp est inscriptible, borné
    "--memory", "256m",               # plafond mémoire
    "--pids-limit", "64",             # plafond de processus
    "--user", "65534:65534",          # exécution en tant que nobody
]
```

> [!info] L'image est épinglée par digest, pas par tag
> `@sha256:...` garantit d'exécuter exactement la couche vérifiée, sans dérive possible d'une reconstruction du tag `python:3.12-slim` à une date ultérieure.

```python
import subprocess, uuid
from dataclasses import dataclass

@dataclass
class ResultatSandbox:
    succes: bool
    sortie: str
    erreur: str

def executer(code: str, delai: int = 10) -> ResultatSandbox:
    """Exécute du code Python dans un conteneur durci, jeté après usage."""
    nom = f"sandbox-{uuid.uuid4().hex[:12]}"
    commande = [
        "docker", "run", "--rm", "--name", nom,
        *DURCISSEMENT, IMAGE, "python", "-c", code,
    ]
    try:
        proc = subprocess.run(commande, capture_output=True, text=True, timeout=delai)
    except subprocess.TimeoutExpired:
        subprocess.run(["docker", "rm", "-f", nom], capture_output=True)
        return ResultatSandbox(False, "", f"délai dépassé ({delai} s)")
    return ResultatSandbox(proc.returncode == 0, proc.stdout, proc.stderr)
```

> [!warning] Le délai est appliqué côté client, pas seulement dans le conteneur
> `--rm` et le nom unique rendent le conteneur jetable — créé pour une exécution, détruit aussitôt, aucun état ne survit. En cas de dépassement du délai, le conteneur est tué explicitement (`docker rm -f`) pour ne pas laisser un processus tourner indéfiniment en arrière-plan.

## Le durcissement, option par option

Chaque option ferme une voie d'évasion précise — **aucune ne suffit seule**, c'est leur cumul qui isole :

- **`--network none`** retire toute interface réseau sauf la boucle locale : le code ne peut ni exfiltrer de données, ni télécharger une charge utile, ni atteindre le réseau interne. C'est souvent la mesure la plus importante.
- **`--cap-drop ALL`** combiné à **`--security-opt no-new-privileges`** retire toutes les capabilities Linux et interdit d'en regagner : le code s'exécute sans aucun pouvoir privilégié.
- **`--read-only`** + **`--tmpfs /tmp:size=16m`** : la racine est non inscriptible, seul `/tmp` rouvre en écriture, et borné — le code peut écrire un fichier temporaire, rien de plus, pas au point de saturer le disque.
- **`--memory`** et **`--pids-limit`** plafonnent mémoire et nombre de processus : une boucle qui alloue sans fin ou une bombe à fork est arrêtée net.
- **`--user 65534:65534`** exécute le code en tant que `nobody`, jamais root, même à l'intérieur du conteneur.

> [!warning] Un conteneur durci partage le noyau de l'hôte
> Ces options réduisent fortement la surface d'attaque, mais une faille du noyau Linux reste une voie d'évasion théorique. Pour du code réellement non fiable, ou un agent exposé publiquement, il faut une couche de plus : gVisor ou une microVM. Le conteneur durci est un excellent défaut, pas un absolu.

## Confiner du code généré par un LLM

Le bac à sable prend tout son sens face à du code qu'on n'a pas écrit :

```python
code = generer_code("Affiche la somme des entiers de 1 à 100.")
resultat = executer(code)
print("Succès :", resultat.succes, "| Sortie :", resultat.sortie.strip())
# >>> Succès : True | Sortie : 5050
```

> [!tip] La réponse d'un modèle arrive souvent enrobée de Markdown
> Les balises ` ``` ` autour du code doivent être retirées avant exécution — un détail, mais un script lancé avec ses balises échouerait dès la compilation.

Tester ce confinement porte sur cinq points : l'extraction du code (déterministe, sans modèle), l'exécution d'un code anodin, le **blocage du réseau** (une tentative d'accès doit échouer — le test le plus parlant, il prouve que `--network none` fait son office), l'application du délai sur une boucle infinie, et l'exécution d'un script réellement écrit par le modèle. Même discipline de test que pour tout agent (voir [[Agents — Pièges classiques]], Piège 8) : la majorité de ces contrôles ne dépendent d'aucun LLM.

## Brancher le bac à sable à un agent

Deux voies, selon le framework :

Avec **smolagents** (voir [[Manques]]), l'isolation est un paramètre : le `CodeAgent` accepte un `executor_type` — `"docker"`, `"e2b"` ou `"modal"` envoient le code généré vers un bac à sable au lieu de l'interpréteur local. L'intégration la plus directe, une ligne de configuration.

Avec un agent maison (la boucle écrite à la main, voir [[Agents 02 — Architecture d'un agent]]), le bac à sable devient un outil comme un autre : là où l'agent appelait une fonction Python directement, il appelle `executer(code)` — le code transite par le conteneur, seul son résultat revient dans la boucle.

> [!info] Le principe constant : séparer la décision de l'exécution
> L'agent décide quel code lancer ; le bac à sable décide ce que ce code a le droit de faire. L'agent garde le contrôle de la logique, l'exécution elle-même reste confinée — la même séparation que la validation Pydantic d'un appel d'outil dans [[LiteLLM 06 — Function calling et outils]], appliquée cette fois à du code arbitraire plutôt qu'à un appel structuré.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| `permission denied` sur le socket Docker | Utilisateur hors du groupe `docker` | Ajouter l'utilisateur au groupe, rouvrir la session — voir [[Docker 08 — Sécurité des conteneurs]] |
| Le conteneur reste actif après un délai | Client tué sans nettoyage | Tuer le conteneur par son nom (`docker rm -f`) |
| Le code échoue à écrire un fichier | Racine en lecture seule | Écrire dans `/tmp` (monté en tmpfs) |
| Un accès réseau attendu échoue | `--network none` | Si le réseau est réellement requis, restreindre plutôt que couper entièrement |
| `OOMKilled` dans les logs | Plafond mémoire atteint | Relever `--memory` si la tâche le justifie |

## Pour aller plus loin

Ce guide couvre l'isolation à l'exécution ; la validation des arguments avant l'exécution (schéma, Pydantic) reste un principe distinct et complémentaire, voir [[Agents 03 — Les outils (Tools)]] et [[LiteLLM 06 — Function calling et outils]].

Sources : [Sandboxing du code des agents — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/agents-sandbox/)
