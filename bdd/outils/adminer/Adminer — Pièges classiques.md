#bdd #adminer #pièges #erreurs #debugging

## 🪤 Piège 1 — Laisser `adminer.php` accessible sans restriction après usage

```bash
# ❌ Déposé pour un dépannage ponctuel, jamais retiré
/var/www/html/adminer.php
```

> [!warning] Une instance oubliée est une porte d'entrée
> Les scanners automatisés testent en continu les chemins `adminer.php` sur les serveurs exposés — combiné à l'historique de CVE réelles (SSRF, lecture de fichier arbitraire), une instance oubliée et non protégée est un risque direct pour le serveur, pas seulement pour la base. Voir [[Adminer 03 — Sécurisation]].

---

## 🪤 Piège 2 — Garder le nom de fichier par défaut en production

```bash
# ❌ Premier chemin testé par les bots
adminer.php
```

```bash
# ✅ Nom imprévisible, en plus des autres protections
admin-db-x7f2k9.php
```

> [!warning] Le renommage seul ne suffit pas
> C'est une mesure complémentaire, pas un remplacement de la restriction par IP et de l'authentification serveur. Voir [[Adminer 03 — Sécurisation]].

---

## 🪤 Piège 3 — Exposer le port Docker sur toutes les interfaces

```yaml
# ❌ Accessible depuis n'importe quelle interface réseau de l'hôte
ports:
  - "8080:8080"
```

```yaml
# ✅ Restreint à l'hôte local uniquement
ports:
  - "127.0.0.1:8080:8080"
```

> [!warning] `internal: true` sur le réseau Docker ne protège pas ce port
> C'est la déclaration `ports:` qui détermine l'exposition externe, indépendamment de la configuration réseau interne du compose. Voir [[Adminer 04 — Déploiement (nginx, Docker, reverse proxy)]].

---

## 🪤 Piège 4 — Se connecter avec un compte à tous les privilèges pour une tâche ponctuelle

```sql
-- ❌ Connexion Adminer avec le compte root de l'instance pour consulter une table
mysql -u root -p
```

> [!warning] Une session Adminer compromise hérite des privilèges du compte utilisé
> Un compte applicatif à droits limités réduit l'impact d'un vol de session ou d'une injection réussie ailleurs dans la chaîne. Voir [[Adminer 03 — Sécurisation]].

---

## 🪤 Piège 5 — Compter sur Adminer pour une stratégie de sauvegarde

```
❌ Export SQL manuel via l'interface web comme unique sauvegarde d'une base de production
```

> [!warning] Pas d'automatisation, pas de cohérence garantie sur une base active
> Contrairement à `mysqldump --single-transaction` ou `pg_dump`, un export manuel via navigateur n'est ni planifié ni testé automatiquement. Voir [[Adminer 02 — Import & export de données]] et [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]] pour une vraie stratégie de sauvegarde.

---

## 🪤 Piège 6 — Laisser une version ancienne en place « parce que ça marche »

```
❌ Instance Adminer 4.6.2 en production, jamais mise à jour depuis son déploiement
```

> [!warning] Des CVE critiques ont été corrigées entre les versions
> Lecture de fichier arbitraire, SSRF activement exploitée (catalogue CISA), XSS : plusieurs failles majeures ont des correctifs disponibles depuis longtemps. Vérifier la version via l'écran de connexion et suivre `adminer.org`. Voir [[Adminer 03 — Sécurisation]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `adminer.php` oublié en ligne après usage | Retirer le fichier après chaque intervention ponctuelle |
| Nom de fichier par défaut conservé | Renommer, en complément des autres protections |
| Port Docker exposé sur toutes les interfaces | Publier sur `127.0.0.1` ou passer par un reverse proxy |
| Connexion avec un compte à tous les privilèges | Utiliser un compte applicatif à droits limités |
| Export manuel utilisé comme sauvegarde de production | Utiliser les outils natifs du moteur, planifiés et testés |
| Version ancienne jamais mise à jour | Suivre les versions et CVE publiées sur `adminer.org` |
