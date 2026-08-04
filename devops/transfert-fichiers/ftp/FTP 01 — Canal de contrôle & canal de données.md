#devops #ftp #fondamentaux

## Deux connexions TCP distinctes pour une seule session

Une session FTP ouvre systématiquement deux connexions séparées :

| Canal | Port standard | Contenu |
|-------|----------------|---------|
| Contrôle | 21 | Commandes (`USER`, `PASS`, `LIST`, `RETR`...) et codes de réponse |
| Données | 20 (mode actif) ou port dynamique (mode passif) | Le contenu réel du fichier transféré, ou le résultat d'un `LIST` |

```
Client                          Serveur
  |──── connexion TCP:21 ───────►|   canal de contrôle (reste ouvert toute la session)
  |◄──── 220 Bienvenue ──────────|
  |──── USER alice ─────────────►|
  |──── PASS ****** ────────────►|
  |◄──── 230 Connecté ───────────|
  |──── LIST ────────────────────►|
  |◄═══ nouvelle connexion :20 ══►|   canal de données (ouvert puis refermé pour CE transfert)
```

> [!info] Le canal de contrôle reste ouvert, le canal de données est éphémère
> Le canal de contrôle persiste pendant toute la durée de la session (jusqu'à `QUIT` ou une déconnexion). Le canal de données, lui, s'ouvre juste avant chaque transfert (un fichier, ou une liste de répertoire) et se referme immédiatement après — une nouvelle connexion est établie pour le transfert suivant.

## Pourquoi cette séparation historique

En 1985, FTP était conçu à une époque où les liaisons réseau étaient lentes et peu fiables : séparer les commandes (qui doivent rester réactives) du transfert de données volumineuses (qui peut être long) évitait qu'un gros transfert ne bloque la capacité à envoyer une commande d'annulation ou de contrôle d'état.

> [!warning] C'est aussi la source de (presque) tous les problèmes pratiques de FTP
> Cette architecture à deux connexions est rare parmi les protocoles applicatifs modernes (HTTP, SSH n'en utilisent qu'une seule). Elle implique que le canal de données doit s'ouvrir *quelque part* — question qui n'a pas de réponse simple dès qu'un pare-feu ou une traduction d'adresse (NAT) s'interpose entre client et serveur. C'est précisément ce que règlent les deux modes de fonctionnement de FTP, actif et passif — voir [[FTP 02 — Mode actif vs mode passif]].

## Le canal de contrôle : un protocole texte simple

Les commandes et réponses du canal de contrôle sont du texte brut, une ligne par échange, avec un code numérique à trois chiffres en préfixe de chaque réponse serveur (repris du modèle SMTP) :

| Plage de code | Signification |
|-----------------|----------------|
| 1xx | Réponse préliminaire positive (action en cours) |
| 2xx | Réponse positive complète (action terminée avec succès) |
| 3xx | Positive intermédiaire (informations supplémentaires attendues) |
| 4xx | Négative temporaire (réessayer plus tard) |
| 5xx | Négative permanente (commande rejetée définitivement) |

```
230 Connexion réussie          # 2xx : succès
331 Mot de passe requis        # 3xx : étape intermédiaire, en attente de PASS
530 Authentification échouée   # 5xx : rejet définitif
```

## Pour aller plus loin

Comment le canal de données s'ouvre réellement — et pourquoi le mode passif s'est imposé face aux pare-feu modernes — fait l'objet de [[FTP 02 — Mode actif vs mode passif]].

Sources : [Active Vs. Passive FTP Simplified — JSCAPE](https://www.jscape.com/blog/active-v-s-passive-ftp-simplified), [File Transfer Protocol — Wikipedia](https://en.wikipedia.org/wiki/File_Transfer_Protocol)
