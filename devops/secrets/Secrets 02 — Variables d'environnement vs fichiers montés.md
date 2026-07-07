#devops #secrets #fondamentaux

## Deux façons d'injecter un secret à l'exécution

| | Variable d'environnement | Fichier monté |
|---|---------------------------|----------------|
| Visible via `docker inspect` | ✅ oui (dans `Config.Env`) | ❌ non |
| Visible via `/proc/<pid>/environ` | ✅ oui, pour tout processus avec les droits suffisants | ❌ non |
| Hérité par les processus enfants | ✅ oui, automatiquement | ❌ non (doit être lu explicitement) |
| Support natif par la quasi-totalité des images | ✅ oui | Dépend de l'image (convention `_FILE`) |
| Rotation sans redémarrer le conteneur | ❌ non (figé au démarrage du process) | ✅ possible si l'application relit le fichier |

```bash
# Variable d'environnement — simple, mais visible dans l'inspection du conteneur
docker run -e DB_PASSWORD=hunter2 myapp

docker inspect myapp --format '{{.Config.Env}}'
# [... "DB_PASSWORD=hunter2" ...]   ← le secret apparaît en clair
```

```bash
# Fichier monté — le secret n'entre jamais dans l'environnement du process
docker run -v ./db_password.txt:/run/secrets/db_password:ro myapp
```

## La convention `_FILE`

Beaucoup d'images officielles (MariaDB, PostgreSQL, WordPress) acceptent une variante `_FILE` de leurs variables d'environnement habituelles, qui pointe vers un fichier à lire au démarrage plutôt que vers la valeur en clair :

```yaml
environment:
  - MARIADB_ROOT_PASSWORD_FILE=/run/secrets/db_root_password   # ✅ lu depuis un fichier
  # au lieu de :
  # - MARIADB_ROOT_PASSWORD=hunter2                             # ❌ en clair
```

Cette convention est ce qui permet aux secrets Docker Compose (voir [[Docker 07 — Docker Compose]]) de s'intégrer sans modification du code applicatif : le secret est monté en fichier sous `/run/secrets/`, et l'image sait où aller le lire.

## Cas particuliers

> [!warning] Toute image ne supporte pas `_FILE`
> Une application qui ne lit que `os.environ` ne saura pas exploiter un fichier monté sans modification de son code. Vérifier la documentation de l'image avant de supposer le support de `_FILE`.

> [!tip] Fichier monté par défaut en production
> Sauf contrainte de compatibilité, préférer un fichier monté à une variable d'environnement pour tout secret réellement sensible (mot de passe de base de données, clé privée) — la variable d'environnement reste acceptable pour des identifiants moins critiques ou en environnement de développement.

> [!info] Ce que ça ne résout pas
> Un fichier monté protège de l'inspection du conteneur, mais pas d'un accès root à la machine hôte ni d'une lecture du fichier source avant montage. Voir [[Secrets 06 — Gestionnaires de secrets externes]] pour un contrôle d'accès plus fin (chiffrement, audit, expiration).
