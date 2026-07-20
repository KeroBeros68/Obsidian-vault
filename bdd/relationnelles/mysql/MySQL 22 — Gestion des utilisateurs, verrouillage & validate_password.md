#bdd #mysql #sécurité #avancé

## CREATE USER avec restriction par host

Contrairement à PostgreSQL, qui contrôle l'accès par IP dans `pg_hba.conf`, MySQL intègre la restriction réseau directement dans le compte utilisateur via la partie `@host` :

```sql
-- Connexion locale uniquement
CREATE USER 'admin_app'@'localhost'
  IDENTIFIED BY 'AdminApp2026!Secure';

-- Connexion depuis un réseau applicatif
CREATE USER 'app_writer'@'10.0.1.%'
  IDENTIFIED BY 'AppWriter2026!Prod';

-- Connexion depuis une IP précise
CREATE USER 'monitoring'@'10.0.1.50'
  IDENTIFIED BY 'Monitor2026!Read';
```

| Pattern `@host` | Signification | Usage |
|------------------|----------------|-------|
| `'localhost'` | Socket Unix + 127.0.0.1 | Admin, CLI local |
| `'10.0.1.%'` | Tout le sous-réseau 10.0.1.x | Réseau applicatif |
| `'10.0.1.50'` | Une seule IP | Monitoring, service spécifique |
| `'%'` | N'importe quelle IP | Dangereux, à éviter |

MySQL évalue `user@host` comme un couple unique : `'alice'@'localhost'` et `'alice'@'10.0.1.%'` sont deux comptes distincts, avec des mots de passe et des privilèges potentiellement différents.

> [!warning] `@'%'` : la règle la plus dangereuse
> Créer un utilisateur avec `@'%'` l'autorise à se connecter depuis n'importe quelle IP — l'équivalent d'un `host all all 0.0.0.0/0` dans `pg_hba.conf`. N'utilisez jamais `@'%'` pour un compte avec des privilèges d'écriture ou d'administration.

## ALTER USER : mot de passe et plugin

```sql
-- Changer le mot de passe
ALTER USER 'app_writer'@'10.0.1.%'
  IDENTIFIED BY 'NouveauMDP2026!Fort';

-- Migrer un compte de mysql_native_password vers caching_sha2_password
ALTER USER 'ancien_compte'@'localhost'
  IDENTIFIED WITH caching_sha2_password BY 'NouveauMDP2026!';
```

Pour lister les comptes, leur plugin, et leur état :

```sql
SELECT user, host, plugin, password_expired, account_locked
FROM mysql.user
WHERE user NOT IN ('mysql.sys','mysql.session','mysql.infoschema')
ORDER BY user, host;
```

## Verrouillage de compte et expiration du mot de passe

```sql
-- Verrouiller un compte (la connexion est refusée)
ALTER USER 'alice'@'localhost' ACCOUNT LOCK;

-- Déverrouiller
ALTER USER 'alice'@'localhost' ACCOUNT UNLOCK;

-- Forcer le changement de mot de passe à la prochaine connexion
ALTER USER 'alice'@'localhost' PASSWORD EXPIRE;

-- Expiration automatique tous les 90 jours
ALTER USER 'alice'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;

-- Historique des mots de passe (empêche la réutilisation)
ALTER USER 'alice'@'localhost' PASSWORD HISTORY 5;

-- Nombre maximum de tentatives échouées avant verrouillage
CREATE USER 'bob'@'localhost'
  IDENTIFIED BY 'Bob2026!Secure'
  FAILED_LOGIN_ATTEMPTS 3
  PASSWORD_LOCK_TIME 1;  -- déverrouillage automatique après 1 jour
```

## `validate_password` : imposer la complexité

Le composant `validate_password` est généralement installé par `mysql_secure_installation` (voir [[MySQL 00 — Installation]]).

```sql
SHOW VARIABLES LIKE 'validate_password%';
```

| Politique | Exigences |
|-----------|-----------|
| `LOW` | Longueur minimale uniquement |
| `MEDIUM` | Longueur + majuscule + minuscule + chiffre + caractère spécial |
| `STRONG` | `MEDIUM` + pas dans un dictionnaire de mots de passe courants |

Pour passer en `STRONG` :

```sql
SET PERSIST validate_password.policy = STRONG;
SET PERSIST validate_password.length = 12;
```

## Pour aller plus loin

Une fois les comptes créés et sécurisés, l'attribution des privilèges suit le principe du moindre privilège via les rôles — voir [[MySQL 23 — Rôles, privilèges granulaires & moindre privilège]].

Sources : [Sécuriser MySQL : utilisateurs, TLS, rôles et moindre privilège — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/securisation/)
