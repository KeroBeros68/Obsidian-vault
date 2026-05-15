#ia #mcp #primitives #tools #resources #prompts #bases

## Les primitives MCP

Un serveur MCP peut exposer 3 types de capacités appelées **primitives** : Tools, Resources et Prompts.

## Primitive 1 — Tools (Outils)

Les fonctions que le LLM peut **appeler pour agir**.

> Équivalent aux outils/tools des agents IA, mais standardisés via MCP.

### Structure d'un Tool

```json
{
  "name": "create_calendar_event",
  "description": "Crée un événement dans Google Calendar. Utilise cet outil quand l'utilisateur veut planifier une réunion, un rappel ou tout événement daté.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "title": {
        "type": "string",
        "description": "Titre de l'événement"
      },
      "start_time": {
        "type": "string",
        "description": "Heure de début au format ISO 8601 (ex: 2025-03-15T14:00:00)"
      },
      "duration_minutes": {
        "type": "integer",
        "description": "Durée en minutes"
      },
      "attendees": {
        "type": "array",
        "items": {"type": "string"},
        "description": "Liste des emails des participants"
      }
    },
    "required": ["title", "start_time"]
  }
}
```

### Exemples de Tools courants

```
Gmail MCP      : send_email, read_email, search_emails, create_draft
GitHub MCP     : create_issue, list_prs, merge_pr, read_file
Fichiers MCP   : read_file, write_file, list_directory, search_files
Slack MCP      : send_message, list_channels, read_messages
PostgreSQL MCP : execute_query, list_tables, describe_table
```

---

## Primitive 2 — Resources (Ressources)

Les **données** que le LLM peut lire (lecture seule, pas d'action).

> Analogie : les Tools sont des verbes (faire), les Resources sont des noms (lire).

### Structure d'une Resource

```json
{
  "uri": "file:///home/user/documents/rapport_q1.pdf",
  "name": "Rapport Q1 2025",
  "description": "Rapport financier du premier trimestre 2025",
  "mimeType": "application/pdf"
}
```

### Types de Resources

```
Statiques  : un fichier, un document, une image
Dynamiques : données temps réel (logs en cours, métriques live)
Templates  : URI avec variables → file:///{chemin}
```

### Exemples de Resources

```
Fichiers MCP    : file:///home/user/notes.md
GitHub MCP      : github://repo/owner/README.md
Base de données : db://ma_base/table/clients
Logs            : logs://application/today
```

### Resources vs Tools

| | Resources | Tools |
|---|---|---|
| **Nature** | Données à lire | Actions à exécuter |
| **Direction** | Serveur → LLM | LLM → Serveur → résultat |
| **Exemples** | Lire un fichier, accéder à des données | Créer un ticket, envoyer un email |
| **Risque** | Faible (lecture seule) | Variable (peut modifier l'état) |

---

## Primitive 3 — Prompts (Templates)

Des **templates de prompts réutilisables** exposés par le serveur, que l'utilisateur peut invoquer.

> Permettent d'encapsuler des workflows complexes dans une commande simple.

### Structure d'un Prompt

```json
{
  "name": "analyser_pr",
  "description": "Analyse une Pull Request GitHub et génère un rapport de review",
  "arguments": [
    {
      "name": "pr_number",
      "description": "Le numéro de la PR à analyser",
      "required": true
    },
    {
      "name": "focus",
      "description": "Aspect à privilégier : 'security', 'performance', 'style'",
      "required": false
    }
  ]
}
```

### Ce que le serveur retourne quand le prompt est invoqué

```json
{
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "Analyse la PR #42 du repo owner/repo. Voici le diff : [diff récupéré automatiquement]. Focus sur la sécurité. Identifie les vulnérabilités potentielles, les bonnes pratiques manquantes, et propose des améliorations concrètes."
      }
    }
  ]
}
```

### Exemples de Prompts MCP

```
GitHub MCP    : "analyser_pr(42)", "générer_release_notes(v2.0)"
Fichiers MCP  : "résumer_document(rapport.pdf)"
Gmail MCP     : "rédiger_réponse(email_id, ton='professionnel')"
SQL MCP       : "analyser_performance_requête(query)"
```

---

## Récapitulatif des 3 primitives

```
SERVEUR MCP
│
├── Tools      → Le LLM FAIT quelque chose
│               (créer, envoyer, modifier, supprimer)
│
├── Resources  → Le LLM LIT quelque chose
│               (fichier, données, document)
│
└── Prompts    → Le LLM utilise un TEMPLATE
                (workflow prêt à l'emploi)
```

> [!tip] Dans la pratique
> La majorité des serveurs MCP exposent principalement des **Tools**. Les Resources et Prompts sont plus rares mais très puissants pour les cas d'usage avancés.

> [!warning] La description des Tools est critique
> Le LLM choisit quel Tool appeler uniquement en lisant sa description. Une description vague → mauvais choix ou non-utilisation. Soigne chaque description comme un prompt.
