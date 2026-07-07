#devops #secrets #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Secret** | Donnée dont la seule divulgation suffit à obtenir un accès non autorisé (mot de passe, clé d'API, jeton, clé privée). |
| **Fichier monté (secret)** | Secret fourni à un conteneur sous forme de fichier (ex. `/run/secrets/<nom>`), plutôt que comme variable d'environnement. |
| **Convention `_FILE`** | Variante d'une variable d'environnement (ex. `MARIADB_ROOT_PASSWORD_FILE`) qui pointe vers un fichier à lire plutôt que vers la valeur en clair. |
| **Docker secret (Swarm)** | Secret chiffré au repos et distribué automatiquement par un cluster Docker Swarm. |
| **Docker secret (Compose)** | Fichier local monté en lecture seule sous `/run/secrets/`, sans chiffrement propre à Compose. |
| **`.dockerignore`** | Fichier listant ce qui doit être exclu du contexte envoyé au démon Docker lors d'un build, évitant qu'un fichier sensible soit copié par erreur. |
| **`docker history`** | Commande révélant les métadonnées de chaque couche d'une image, y compris les valeurs `ARG`/`ENV` utilisées au build. |
| **`RUN --mount=type=secret`** | Mécanisme BuildKit montant un secret uniquement pendant l'exécution d'un `RUN`, sans le persister dans l'image. |
| **Masquage (CI)** | Remplacement automatique d'un secret par `***` dans les logs CI dès qu'il apparaît littéralement en sortie. |
| **OIDC (identité fédérée)** | Mécanisme d'authentification par jeton de courte durée entre une plateforme CI et un fournisseur cloud, sans secret statique stocké. |
| **Secret statique** | Identifiant valide jusqu'à révocation manuelle explicite. |
| **Secret dynamique** | Identifiant généré à la demande avec une durée de vie limitée, expirant automatiquement (ex. via HashiCorp Vault). |
| **HashiCorp Vault** | Gestionnaire de secrets agnostique du cloud, capable de générer des secrets dynamiques via des moteurs dédiés (bases de données, PKI, cloud). |
| **Rotation de secrets** | Remplacement périodique ou automatisé d'un secret pour limiter la fenêtre d'exploitation en cas de fuite. |
| **Blast radius** | Étendue de l'impact potentiel d'un secret compromis — réduite par des secrets à portée restreinte et à courte durée de vie. |
| **`--mount=type=secret,required=true`** | Option faisant échouer le build si le secret attendu n'est pas fourni, plutôt que de continuer silencieusement sans lui. |
| **`--mount=type=secret,env=`** | Option injectant un secret BuildKit directement comme variable d'environnement dans le `RUN`, sans fichier intermédiaire. |
| **Compose build secrets** | Secrets transmis à `docker build` (et non au conteneur en exécution) via la clé `build.secrets` d'un service Compose. |
| **SOPS** | Outil (Mozilla) chiffrant un fichier YAML/JSON/ENV champ par champ (clés lisibles, valeurs chiffrées), pour le committer en sécurité dans Git — sans dépendre d'un service externe au runtime. |
| **trufflehog** | Outil de détection de secrets dans un historique Git ou une image de conteneur déjà construite. |
| **Gitleaks** | Outil de détection de secrets exécuté en hook pre-commit, bloquant un commit avant qu'il n'atteigne le dépôt. |
