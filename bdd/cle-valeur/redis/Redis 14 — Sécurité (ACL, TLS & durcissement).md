#bdd #redis #sécurité #avancé

## Redis n'est pas sécurisé par défaut

Historiquement, Redis n'exigeait aucune authentification par défaut, en présumant un déploiement derrière un pare-feu. Une instance exposée sans mot de passe sur Internet est immédiatement compromise — un scénario d'attaque massivement documenté et toujours d'actualité.

```bash
# redis.conf — un minimum non négociable
bind 127.0.0.1 10.0.1.5
requirepass "MotDePasseFort2026!"
protected-mode yes
```

## ACL : des comptes nommés avec des permissions fines

Depuis Redis 6.0, l'authentification par simple mot de passe (`requirepass`) peut être remplacée par un système d'**ACL** (*Access Control List*), avec des comptes nommés et des permissions par commande, par catégorie et par motif de clé.

```bash
ACL SETUSER app_lecture on >MotDePasse2026! ~cache:* +@read
ACL SETUSER app_ecriture on >AutreMotDePasse! ~cache:* ~session:* +@read +@write -@dangerous
ACL LIST
ACL WHOAMI
```

| Élément | Rôle |
|---------|------|
| `on` / `off` | Active ou désactive le compte |
| `>motdepasse` | Définit un mot de passe (plusieurs possibles par compte) |
| `~motif` | Restreint l'accès aux clés correspondant au motif (`~cache:*`) |
| `+@categorie` | Autorise une catégorie de commandes (`@read`, `@write`, `@admin`...) |
| `-@categorie` | Interdit explicitement une catégorie (`-@dangerous` exclut `FLUSHALL`, `CONFIG`, `SHUTDOWN`...) |
| `+commande` | Autorise une commande précise, indépendamment de sa catégorie |

> [!warning] Toujours exclure @dangerous des comptes applicatifs
> La catégorie `@dangerous` regroupe des commandes capables de vider la base (`FLUSHALL`, `FLUSHDB`), de modifier la configuration à chaud (`CONFIG SET`) ou d'arrêter le serveur (`SHUTDOWN`). Un compte applicatif standard ne devrait jamais en avoir besoin.

## TLS : chiffrer le transport

Redis ne chiffre pas ses échanges par défaut. Le support TLS s'active au niveau du serveur :

```bash
# redis.conf
tls-port 6380
port 0                          # Désactive le port en clair
tls-cert-file /etc/redis/ssl/redis.crt
tls-key-file /etc/redis/ssl/redis.key
tls-ca-cert-file /etc/redis/ssl/ca.crt
```

```bash
redis-cli --tls --cert client.crt --key client.key --cacert ca.crt -h redis.exemple.com PING
```

> [!info] Authentification mutuelle par certificat depuis Redis 8.6
> Redis 8.6 a introduit l'authentification automatique par certificat client TLS, permettant d'identifier un compte ACL directement à partir du certificat présenté, sans mot de passe supplémentaire à gérer côté application.

## Renommer ou désactiver les commandes sensibles

```bash
# redis.conf
rename-command FLUSHALL ""
rename-command CONFIG "config_9f3a7b"
```

Utile en complément des ACL sur une instance qui doit rester accessible à des outils d'administration externes sans exposer ses commandes les plus destructrices.

## Checklist de durcissement

| Vérification | Commande | Attendu |
|----------------|----------|---------|
| Pas d'écoute sur toutes les interfaces sans pare-feu | `CONFIG GET bind` | Pas `0.0.0.0` sans pare-feu en complément |
| `protected-mode` actif | `CONFIG GET protected-mode` | `yes` |
| Authentification active (mot de passe ou ACL) | `CONFIG GET requirepass` / `ACL LIST` | Non vide / comptes définis |
| TLS activé si trafic sur réseau non fiable | `CONFIG GET tls-port` | Port TLS configuré |
| Comptes applicatifs sans `@dangerous` | `ACL GETUSER app_ecriture` | `-@dangerous` présent |
| Commandes destructrices renommées/désactivées | `CONFIG GET rename-command` | `FLUSHALL`/`CONFIG` restreints en environnement partagé |

## Pour aller plus loin

Cela conclut le module Redis — voir [[Redis — Index des fiches]] pour une vue d'ensemble, ou [[BDD — Home]] pour explorer les autres moteurs (MySQL, MariaDB, SQL).

Sources : [ACL — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/), [Redis 8.6 — What's New — VersionLog](https://versionlog.com/redis/8.6/)
