#bdd #mariadb #sécurité #avancé

## unix_socket : root sans mot de passe, mais pas sans sécurité

Contrairement à MySQL (`caching_sha2_password` par défaut, voir [[MySQL 21 — Authentification (caching_sha2_password & authentication_policy)]]), le compte `root@localhost` de MariaDB s'authentifie par défaut via le plugin **`unix_socket`**, installé et actif d'origine.

```sql
SELECT User, Host, plugin FROM mysql.global_priv WHERE User = 'root';
```

Le principe : le plugin récupère l'identité de l'utilisateur du système d'exploitation connecté via la socket Unix (`SO_PEERCRED`) et authentifie ce même utilisateur OS comme le compte MariaDB correspondant — sans mot de passe.

```bash
# Fonctionne si connecté en tant qu'utilisateur OS "root"
sudo mariadb
```

> [!info] Forces et limites de `unix_socket`
> Force : élimine les attaques par force brute sur le mot de passe root et son exposition dans des scripts. Limite : la sécurité du compte dépend alors entièrement du contrôle d'accès du système d'exploitation — un terminal laissé ouvert, un `sudo` trop permissif, ou une faille d'injection de commande dans une application web compromettent le compte tout autant qu'un mot de passe faible.

## ed25519 : l'alternative moderne pour les comptes réseau

Pour les comptes qui doivent s'authentifier par mot de passe (accès réseau, applications), MariaDB propose **`ed25519`**, basé sur l'algorithme de signature elliptique du même nom — une alternative plus moderne que les schémas basés sur SHA-1.

```sql
INSTALL SONAME 'auth_ed25519';

CREATE USER 'app_writer'@'10.0.1.%'
  IDENTIFIED VIA ed25519 USING PASSWORD('AppWriter2026!Prod');
```

> [!warning] Non installé par défaut
> La bibliothèque du plugin `ed25519` est distribuée avec MariaDB, mais elle n'est **pas installée automatiquement** : `INSTALL SONAME` est requis avant de pouvoir créer un compte l'utilisant.

## mysql_native_password : présent mais déconseillé

`mysql_native_password` est lié statiquement au serveur (aucune installation nécessaire) et reste disponible pour compatibilité avec d'anciens clients ou scripts. Il repose sur un double hash SHA-1, jugé insuffisant pour de nouvelles installations exigeant un haut niveau de sécurité.

```sql
CREATE USER 'legacy_app'@'10.0.1.%'
  IDENTIFIED VIA mysql_native_password USING PASSWORD('MotDePasse2026!');
```

| Plugin | Algorithme | Installation requise | Recommandation |
|--------|-----------|-------------------------|-------------------|
| `unix_socket` | Identité OS | Non (actif par défaut) | Comptes d'administration locaux |
| `ed25519` | EdDSA (courbe elliptique) | Oui (`INSTALL SONAME`) | Comptes réseau, nouvelles installations |
| `mysql_native_password` | SHA-1 double | Non (lié statiquement) | Compatibilité legacy uniquement |

## CREATE USER : la syntaxe IDENTIFIED VIA

MariaDB privilégie une syntaxe `IDENTIFIED VIA plugin USING ...` plus explicite que le `IDENTIFIED WITH plugin BY '...'` de MySQL, bien que ce dernier reste également accepté pour la compatibilité :

```sql
-- Syntaxe MariaDB idiomatique
CREATE USER 'monitoring'@'10.0.1.50' IDENTIFIED VIA ed25519 USING PASSWORD('Monitor2026!');

-- Syntaxe compatible MySQL, également valide
CREATE USER 'monitoring'@'10.0.1.50' IDENTIFIED BY 'Monitor2026!';
```

## Pour aller plus loin

Les réglages de configuration (`my.cnf`, buffers, connexions) sont couverts dans [[MariaDB 09 — Configuration (my.cnf, InnoDB & Aria)]].

Sources : [Authentication Plugin - Unix Socket — MariaDB Documentation](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-unix-socket), [Authentication Plugin - ed25519 — MariaDB Documentation](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-ed25519), [Authentication Plugin - mysql_native_password — MariaDB Documentation](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-mysql_native_password)
