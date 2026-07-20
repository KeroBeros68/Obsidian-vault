#bdd #mysql #sécurité #avancé

## Vérifier l'état TLS actuel

```sql
SHOW VARIABLES WHERE Variable_name IN ('ssl_ca','ssl_cert','ssl_key','tls_version');
```

> [!warning] `have_ssl` et `have_openssl` supprimées en 8.4
> Ces variables n'existent plus en MySQL 8.4. Pour vérifier que TLS est actif, inspecter les variables `ssl_*` ci-dessus et le statut `Ssl_cipher` de la session courante.

```sql
SHOW SESSION STATUS LIKE 'Ssl_cipher';
-- Ssl_cipher = TLS_AES_256_GCM_SHA384  → connexion chiffrée
-- Ssl_cipher vide                      → connexion non chiffrée
```

## Certificats auto-générés par MySQL

Lors de la première initialisation (`mysqld --initialize`, voir [[MySQL 00 — Installation]]), MySQL génère automatiquement des certificats auto-signés dans le datadir :

```
-rw------- 1 mysql mysql 1704 ca-key.pem
-rw-r--r-- 1 mysql mysql 1131 ca.pem
-rw-r--r-- 1 mysql mysql 1131 server-cert.pem
-rw------- 1 mysql mysql 1704 server-key.pem
-rw-r--r-- 1 mysql mysql 1131 client-cert.pem
-rw------- 1 mysql mysql 1704 client-key.pem
```

Ces certificats auto-signés suffisent pour le chiffrement du transport, mais ne permettent pas de vérifier l'identité du serveur côté client. En production, utiliser des certificats signés par une PKI interne.

## Configurer TLS avec des certificats personnalisés

```ini
[mysqld]
ssl_ca = /etc/mysql/ssl/ca.pem
ssl_cert = /etc/mysql/ssl/server-cert.pem
ssl_key = /etc/mysql/ssl/server-key.pem
tls_version = TLSv1.2,TLSv1.3
```

```bash
chown mysql:mysql /etc/mysql/ssl/server-key.pem
chmod 600 /etc/mysql/ssl/server-key.pem
sudo systemctl restart mysql
```

## Forcer TLS dans CREATE USER (REQUIRE SSL / X509)

MySQL permet de forcer le TLS au niveau du compte utilisateur, contrairement à PostgreSQL qui le contrôle dans `pg_hba.conf` :

```sql
-- Forcer TLS (chiffrement uniquement)
CREATE USER 'app_secure'@'10.0.1.%'
  IDENTIFIED BY 'Secure2026!TLS'
  REQUIRE SSL;

-- Forcer TLS avec certificat client (authentification mutuelle)
CREATE USER 'app_mutual'@'10.0.1.%'
  IDENTIFIED BY 'Mutual2026!Cert'
  REQUIRE X509;

-- Sur un compte existant
ALTER USER 'app_writer'@'10.0.1.%' REQUIRE SSL;
```

| `REQUIRE` | Chiffrement | Vérifie certificat client | Usage |
|-----------|-------------|----------------------------|-------|
| `NONE` | Non requis | Non | Défaut, dangereux en production |
| `SSL` | Oui | Non | Minimum recommandé |
| `X509` | Oui | Oui | Automatisation, services critiques |
| `ISSUER '...' SUBJECT '...'` | Oui | Oui + vérifie l'émetteur/sujet | Très restrictif |

Test de connexion avec TLS forcé :

```bash
# Connexion chiffrée — fonctionne
mysql -u app_secure -h 10.0.1.5 -p --ssl-mode=REQUIRED

# Connexion sans TLS — échoue
mysql -u app_secure -h 10.0.1.5 -p --ssl-mode=DISABLED
# ERROR 1045 (28000): Access denied for user 'app_secure'@'10.0.1.5'
```

## Chiffrement vs vérification d'identité (côté client)

`REQUIRE SSL` sur le compte force le chiffrement du transport, mais ne vérifie pas que le client se connecte au bon serveur. Côté client, le paramètre `--ssl-mode` contrôle le niveau de vérification :

| `--ssl-mode` | Chiffrement | Vérifie le certificat serveur | Vérifie le hostname | Usage |
|--------------|-------------|-------------------------------|----------------------|-------|
| `DISABLED` | Non | Non | Non | Interdit en production |
| `PREFERRED` | Si possible | Non | Non | Défaut, chiffre si le serveur le supporte |
| `REQUIRED` | Oui | Non | Non | Force le chiffrement, pas la vérification |
| `VERIFY_CA` | Oui | Oui | Non | Vérifie que le certificat est signé par une CA de confiance |
| `VERIFY_IDENTITY` | Oui | Oui | Oui | Le plus sûr — vérifie aussi que le hostname correspond au CN/SAN du certificat |

> [!warning] `VERIFY_IDENTITY` et certificats auto-signés
> Les certificats auto-générés par MySQL à l'initialisation ne contiennent pas de *Subject Alternative Name* (SAN) correspondant au hostname du serveur. `--ssl-mode=VERIFY_IDENTITY` échouera avec ces certificats. En production, utiliser des certificats signés par une PKI interne avec le hostname du serveur dans le SAN.

> [!tip] Recommandation de production
> Configurer `REQUIRE SSL` côté serveur et `--ssl-mode=VERIFY_CA` (ou `VERIFY_IDENTITY`) côté client, pour se protéger contre les attaques de type *man-in-the-middle*.

## Recharger les certificats à chaud

```sql
ALTER INSTANCE RELOAD TLS;
SHOW STATUS LIKE 'Ssl_server_not_after';
```

## Pour aller plus loin

Le chiffrement protège le transport ; la restriction du réseau et l'audit des connexions complètent la défense en profondeur — voir [[MySQL 25 — Restreindre le réseau & auditer les connexions]].

Sources : [Sécuriser MySQL : utilisateurs, TLS, rôles et moindre privilège — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/securisation/)
