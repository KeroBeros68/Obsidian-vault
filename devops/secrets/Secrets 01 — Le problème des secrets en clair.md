#devops #secrets #fondamentaux

## Qu'est-ce qu'un secret

Un **secret** est toute donnée qui donne un accès si elle est connue : mot de passe, clé d'API, jeton d'authentification (token), clé privée de certificat, chaîne de connexion à une base de données. Contrairement à une configuration classique (nom d'hôte, port, feature flag), sa seule divulgation a un impact de sécurité direct.

## Où les secrets finissent en clair par erreur

| Erreur courante | Conséquence |
|-------------------|-------------|
| Mot de passe écrit en dur dans le code source | Visible par quiconque a accès au dépôt, y compris après suppression (reste dans l'historique Git) |
| `ENV DB_PASSWORD=...` dans un Dockerfile | Persiste dans les métadonnées de l'image, lisible via `docker history` |
| `.env` non listé dans `.gitignore` | Committé par erreur, souvent dès le tout premier commit d'un projet |
| Secret passé en argument de ligne de commande | Visible par tout utilisateur de la machine via `ps aux` |
| Secret loggé (debug, stack trace) | Persiste dans des fichiers de logs, souvent moins surveillés que le code source |

## Le principe : injection à l'exécution, jamais dans l'artefact

Un secret ne doit jamais faire partie d'un artefact versionné ou construit (code source, image Docker, package) : il doit être **injecté au moment de l'exécution**, depuis une source contrôlée (variable d'environnement fournie par l'orchestrateur, fichier monté, gestionnaire de secrets externe).

```
❌ code source / Dockerfile ──(contient le secret)──> artefact versionné, diffusé, mis en cache
✅ artefact générique ──(sans secret)──> exécution ──(injection)──> secret fourni au dernier moment
```

## Cas particuliers

> [!warning] Git n'oublie rien
> `git rm secret.txt` puis un commit ne supprime **pas** le secret de l'historique — il reste récupérable dans les commits précédents tant que l'historique n'est pas réécrit (`git filter-repo`, BFG Repo-Cleaner). La seule remédiation fiable après une fuite dans un dépôt Git est de **révoquer et régénérer le secret**, pas de nettoyer l'historique.

> [!tip] Penser rotation, pas seulement prévention
> Partir du principe que tout secret finira, un jour, par fuiter (erreur humaine, dépendance compromise, log mal configuré) change la priorité : un secret facile à révoquer et régénérer limite les dégâts bien mieux qu'une prévention supposée parfaite. Voir [[Secrets 06 — Gestionnaires de secrets externes]] pour la rotation automatisée.

> [!info] Secret vs configuration
> Une valeur de configuration (port, nom de service, niveau de log) peut rester en clair sans risque. Le critère de bascule vers "secret" : sa divulgation seule suffit-elle à obtenir un accès non autorisé ?
