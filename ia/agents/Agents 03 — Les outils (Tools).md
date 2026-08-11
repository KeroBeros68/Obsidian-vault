#ia #agents #outils #tools #bases

## Les outils (Tools)

Les outils sont les **actions que l'agent peut exécuter** dans le monde réel. Sans outils, un agent n'est qu'un LLM qui réfléchit sans agir.

## Principe de fonctionnement

Le LLM ne "voit" jamais le code des outils — il voit uniquement leurs **descriptions en langage naturel**. Il décide quel outil appeler et avec quels paramètres.

```
LLM lit les descriptions → Choisit l'outil → Génère les paramètres
                                                      ↓
Orchestrateur → Exécute la vraie fonction Python/JS
                                                      ↓
Résultat → Ajouté au contexte du LLM → LLM continue
```

## Les grandes catégories d'outils

### Recherche et information
```
recherche_web(query)         → résultats de recherche en temps réel
recherche_rag(query)         → documents internes
wikipedia(sujet)             → article Wikipedia
actualites(sujet, date)      → news récentes
```

### Calcul et code
```
executer_python(code)        → exécute du code, retourne le résultat
calculatrice(expression)     → calcul mathématique
analyser_csv(fichier, query) → analyse un fichier de données
```

### Fichiers et documents
```
lire_fichier(chemin)         → lire un fichier local
ecrire_fichier(chemin, texte)→ écrire dans un fichier
lire_pdf(url_ou_chemin)      → extraire le texte d'un PDF
creer_rapport(données)       → générer un document
```

### Communication
```
envoyer_email(dest, sujet, corps)  → Gmail, Outlook...
envoyer_slack(canal, message)      → notification Slack
creer_ticket(titre, desc)          → Jira, Linear...
```

### APIs et services
```
appel_api(url, méthode, params)    → appel HTTP générique
mettre_a_jour_crm(id, données)     → Salesforce, HubSpot...
creer_evenement(titre, date)       → Google Calendar...
requete_sql(query)                 → base de données
```

### Navigation web (agents avancés)
```
ouvrir_page(url)             → charger une page web
cliquer(element)             → cliquer sur un bouton ou lien
remplir_formulaire(champ, valeur) → saisir du texte
faire_screenshot()           → capturer l'écran
```

## Comment bien définir un outil

La description est cruciale — le LLM s'en sert pour décider quand et comment utiliser l'outil.

```python
# ❌ Description trop vague
{
  "name": "search",
  "description": "Fait une recherche"
}

# ✅ Description claire et actionnable
{
  "name": "recherche_web",
  "description": "Cherche des informations récentes sur Internet via Google.
                  Utilise cet outil pour : actualités, prix en temps réel,
                  informations récentes, vérification de faits.
                  Ne pas utiliser pour : les données internes de l'entreprise,
                  les calculs, la lecture de fichiers locaux.",
  "parameters": {
    "query": {
      "type": "string",
      "description": "La requête de recherche. Formule-la comme une vraie recherche Google."
    },
    "nb_resultats": {
      "type": "integer",
      "description": "Nombre de résultats souhaités (1-10). Par défaut : 5.",
      "default": 5
    }
  }
}
```

## Outils natifs des frameworks

| Framework | Outils inclus |
|---|---|
| **LangChain** | Web search, Wikipedia, Python REPL, SQL, fichiers, APIs... |
| **LlamaIndex** | RAG, web search, code interpreter, APIs... |
| **AutoGen** | Code execution, web search, fichiers... |
| **Claude (API)** | Web search natif, computer use (clic, navigation) |

## Sécurité des outils

Classer les outils par niveau de risque et adapter les guardrails.

| Niveau | Type d'outil | Guardrail |
|---|---|---|
| 🟢 Sûr | Lecture seule (recherche, lecture fichier) | Aucun |
| 🟡 Modéré | Écriture réversible (créer un brouillon) | Log de l'action |
| 🔴 Critique | Actions irréversibles (envoyer email, supprimer) | Validation humaine |

> [!warning] Principe du moindre privilège
> Ne donner à l'agent que les outils dont il a réellement besoin. Un agent avec trop d'outils fait plus d'erreurs et est plus difficile à déboguer.

> [!warning] Les secrets vivent hors du code des outils
> Un outil qui appelle une API (CRM, email, base de données) a besoin de clés ou de mots de passe — jamais en clair dans le code de l'outil, jamais commités dans Git. Ils vivent dans des variables d'environnement ou un gestionnaire de secrets dédié. Voir [[Secrets 01 — Le problème des secrets en clair]] et [[Secrets — Index des fiches]] pour les mécanismes.

> [!tip] Commencer simple
> Commence avec 2-3 outils max. Ajoute des outils seulement quand l'agent en a clairement besoin. Un agent avec 15 outils est souvent moins efficace qu'un agent avec 4 outils bien définis.

## Le LLM ne "fait" jamais l'appel, il génère du JSON à valider

Le LLM ne se connecte jamais réellement à une API ni n'exécute de code : il génère un texte structuré (JSON) qui décrit quel outil appeler et avec quels paramètres. C'est le code de l'application qui reçoit ce JSON, doit le **valider**, puis l'exécuter réellement.

> [!tip] Le même modèle Pydantic sert deux fois
> Plutôt qu'écrire à la main le schéma JSON envoyé au modèle et séparément la logique de validation, un seul modèle Pydantic peut produire les deux : `MesArgs.model_json_schema()` génère le schéma décrit au LLM, et `MesArgs(**arguments)` valide ce qu'il renvoie. Les deux restent alors automatiquement synchronisés — voir [[LiteLLM 06 — Function calling et outils]] pour l'exemple complet, schéma généré, validation, gestion des trois pannes réelles (JSON cassé, argument manquant, outil inconnu).

```python
# Le LLM génère ceci (une proposition d'appel, pas une exécution) :
# {"name": "envoyer_email", "arguments": {"destinataire": "'; DROP TABLE users;--", "sujet": "..."}}

# ❌ Exécuter directement sans validation
appeler_outil(nom, arguments)

# ✅ Valider le schéma avant toute exécution
from pydantic import BaseModel, EmailStr

class EnvoyerEmailArgs(BaseModel):
    destinataire: EmailStr
    sujet: str
    corps: str

try:
    args_valides = EnvoyerEmailArgs(**arguments)
    appeler_outil(nom, args_valides)
except ValidationError as e:
    # Retourner une erreur exploitable par le LLM, pas un crash
    return f"Appel invalide : {e}. Réessaie avec un destinataire valide."
```

> [!warning] Un LLM peut générer un appel malformé, jamais malveillant "exprès" — mais l'effet est le même
> Le LLM ne cherche pas activement à attaquer le système, mais rien ne garantit que le JSON généré respecte le schéma attendu (type incorrect, champ manquant, valeur hors énumération) — et un contenu extérieur injecté dans le contexte (voir le *Tool Poisoning*, [[MCP — Pièges classiques]]) peut aussi pousser le LLM à générer un appel dangereux. Valider systématiquement (Pydantic, JSON Schema, ou équivalent) avant d'exécuter quoi que ce soit, et renvoyer une erreur typée et compréhensible plutôt qu'un crash — ce contrat d'interface est la base de tout *structured output* fiable.

> [!tip] Même un type simple peut arriver déformé
> Un modèle renvoie parfois un nombre sous forme de chaîne (`"23"` au lieu de `23`) malgré un schéma qui déclare `number` — sans que ce soit une attaque, juste une imprécision de génération. Un outil qui prend des paramètres numériques a intérêt à coercer explicitement (`float(a)`, `int(b)`) plutôt que de faire confiance aveuglément au type déclaré :
> ```python
> def addition(a: float, b: float) -> float:
>     """Additionne deux nombres."""
>     return float(a) + float(b)  # coercion défensive, même si le schéma promet déjà un nombre
> ```
