#devops #secrets #avancé

## Le problème que Docker/CI ne résolvent pas seuls

Les mécanismes vus jusqu'ici (fichiers montés, secrets CI) répondent à "comment injecter un secret sans le versionner" — mais pas à : qui a eu accès à quel secret et quand, comment le faire tourner (rotation) sans interrompre le service, ni comment limiter sa durée de vie. Les gestionnaires de secrets externes (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault) répondent à ce second niveau.

| Solution | Écosystème | Point fort |
|-----------|------------|-------------|
| HashiCorp Vault | Agnostique (auto-hébergé ou Vault Cloud) | Secrets dynamiques, moteurs pluggables (bases de données, cloud, PKI) |
| AWS Secrets Manager | AWS | Rotation automatique intégrée aux services AWS (RDS...) |
| GCP Secret Manager | Google Cloud | Intégration IAM native |
| Azure Key Vault | Azure | Gestion conjointe secrets / clés / certificats |

## Secrets statiques vs secrets dynamiques

```
Secret statique  : créé une fois → stocké → lu par l'appli → révoqué manuellement si besoin
Secret dynamique : demandé par l'appli → généré à la volée avec une durée de vie limitée → expire automatiquement
```

Vault peut, par exemple, générer un identifiant de connexion PostgreSQL **temporaire** à la demande, avec un TTL de quelques heures — si ce credential fuite, il devient inutilisable de lui-même après expiration, sans intervention manuelle.

```bash
# Exemple conceptuel : demander un credential DB dynamique à Vault
vault read database/creds/readonly-role
# Renvoie un username/password valides seulement pour la durée configurée
```

## SOPS : chiffrer ce qu'on versionne, sans service externe

Les solutions ci-dessus sont des **services** interrogés à l'exécution. **SOPS** (Secrets OPerationS, Mozilla) répond à un besoin différent et plus léger : chiffrer un fichier YAML/JSON/ENV pour pouvoir le **committer dans Git en toute sécurité**, sans dépendre d'un service tiers disponible au runtime.

```bash
# Chiffrer un fichier de secrets (les clés restent lisibles, les valeurs non)
sops -e -i secrets.json
```

```json
{
  "api_key": "ENC[AES256_GCM,data:abc123...,type:str]",
  "db_password": "ENC[AES256_GCM,data:def456...,type:str]"
}
```

```bash
# Déchiffrer côté CI, là où le runner a accès à la clé (GPG ou KMS cloud)
sops -d secrets.json > decrypted.json
docker build --secret id=config,src=decrypted.json -t mon-app .
rm -f decrypted.json
```

SOPS supporte plusieurs backends de clé : une paire GPG classique, ou un service de gestion de clés cloud (`sops --kms arn:aws:kms:...`).

| | SOPS | Vault / AWS / GCP / Azure |
|---|------|----------------------------|
| Ce qui est protégé | Un fichier versionné dans Git | Des secrets stockés et servis par un service |
| Dépendance au runtime | Aucune — seul le déchiffrement initial a besoin de la clé | Le service doit être disponible à chaque lecture |
| Secrets dynamiques (rotation auto, TTL) | ❌ non, le contenu reste statique jusqu'au prochain chiffrement | ✅ oui, pour Vault notamment |
| Cas d'usage typique | Petite équipe, config versionnée avec le code (GitOps) | Secrets partagés à grande échelle, rotation automatique exigée |

> [!tip] SOPS et Vault ne s'excluent pas
> Un dépôt GitOps peut très bien chiffrer avec SOPS des secrets de configuration statiques (peu nombreux, changeant rarement) tout en déléguant à Vault les identifiants dynamiques à forte rotation — les deux répondent à des échelles de criticité différentes plutôt qu'à un même besoin.

## Injection dans un conteneur ou un pod

| Pattern | Fonctionnement |
|----------|-----------------|
| Init container / sidecar (Vault Agent Injector pour Kubernetes) | Récupère les secrets depuis Vault avant/pendant le démarrage du pod, les écrit en fichier local |
| Appel API direct depuis l'application | L'application s'authentifie auprès du gestionnaire et récupère ses secrets au démarrage |
| CSI driver (Secrets Store CSI Driver) | Monte les secrets externes comme un volume Kubernetes natif |

## Cas particuliers

> [!tip] Les secrets dynamiques réduisent le "blast radius"
> Un secret dynamique de courte durée limite l'impact d'une fuite au temps restant avant expiration, sans nécessiter de rotation manuelle. C'est un changement de modèle par rapport à un mot de passe statique qui reste valide tant que personne ne le révoque explicitement.

> [!warning] Un gestionnaire de secrets est lui-même un système critique
> Vault (auto-hébergé) doit être démarré "unsealed" (déverrouillé cryptographiquement) et configuré en haute disponibilité — une panne du gestionnaire de secrets peut bloquer le démarrage de tout ce qui en dépend. Les offres managées (AWS/GCP/Azure) déportent cette charge opérationnelle vers le fournisseur cloud.

> [!info] Prolonge Docker 08
> Cette couche complète les pratiques déjà vues dans [[Docker 08 — Sécurité des conteneurs]] (utilisateur non-root, socket Docker) : limiter les privilèges d'un conteneur ne protège pas les secrets qu'il manipule, ce sont deux axes de sécurité complémentaires.
