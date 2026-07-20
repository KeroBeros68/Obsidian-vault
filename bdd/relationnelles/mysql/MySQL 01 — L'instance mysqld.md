#bdd #mysql #fondamentaux #architecture

## Un processus unique, multi-thread

Contrairement à un modèle multi-processus (un processus par connexion), MySQL utilise un modèle **multi-thread** : un seul processus, `mysqld`, gère tout — il écoute les connexions, exécute les requêtes et coordonne les opérations d'I/O. Chaque client connecté obtient un **thread** dédié à l'intérieur de ce processus unique.

```bash
ps aux | grep mysqld
# mysql  4521  2.3  8.1 1842384 331276 ?  Ssl  08:49  0:12 /usr/sbin/mysqld
```

Un seul PID pour tout : gestion des connexions, du cache, des I/O et de la maintenance se fait via des threads à l'intérieur de ce processus.

> [!tip] Analogie : l'open space vs l'immeuble de bureaux
> Un moteur multi-processus (une connexion = un processus séparé) ressemble à un immeuble de bureaux où chaque employé a son propre bureau. MySQL, avec son modèle multi-thread, ressemble à un open space : tout le monde travaille dans la même pièce (processus), chacun à son poste (thread). Avantage : un thread coûte moins cher à créer qu'un processus. Inconvénient : un problème mémoire dans un thread peut affecter tous les autres, puisqu'ils partagent le même espace processus.

## Le modèle client/serveur

Le serveur (`mysqld`) écoute sur un **socket Unix** (connexion locale) ou un **port TCP** (connexion réseau, par défaut `3306`). Le client (`mysql`, `mysqldump`, `mysqlsh`, ou toute application) s'y connecte, envoie des requêtes SQL et reçoit les résultats.

```bash
# Socket Unix — connexion locale, plus rapide (pas de couche réseau)
ls -la /var/run/mysqld/mysqld.sock

# Port TCP — connexion réseau
ss -tlnp | grep 3306
# LISTEN  0  151  127.0.0.1:3306  0.0.0.0:*  users:(("mysqld",pid=4521,fd=23))
```

| Type | Commande | Usage |
|------|----------|-------|
| Socket Unix | `mysql -u root -p` | Connexion locale, administration — utilisé par défaut sans `-h` |
| TCP localhost | `mysql -h 127.0.0.1 -u root -p` | Connexion locale via TCP |
| TCP réseau | `mysql -h 192.168.122.70 -u labadmin -p` | Connexion depuis un autre serveur |

> [!warning] `bind-address` limite les connexions réseau par défaut
> MySQL n'écoute par défaut que sur `127.0.0.1` — aucune connexion depuis un autre serveur n'est possible tant que `bind-address` n'est pas modifié dans `/etc/mysql/mysql.conf.d/mysqld.cnf`. Un choix par défaut sécurisé, pas une limitation à contourner sans réflexion.

## Cas particuliers

> [!info] `SHOW PROCESSLIST` liste les threads de connexion actifs
> Chaque ligne renvoyée représente un thread de connexion actuellement ouvert. Le nombre maximal simultané est contrôlé par `max_connections` (défaut 151) — au-delà, MySQL refuse toute nouvelle connexion avec l'erreur `Too many connections`.

## Pour aller plus loin

Les bases de données et le modèle client/serveur en détail dans [[MySQL 02 — Bases de données, schémas & bases système]] ; l'architecture interne complète (parseur, optimiseur...) dans [[MySQL 03 — Architecture interne (les couches de mysqld)]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
