#bdd #redis #pièges #erreurs #debugging

## 🪤 Piège 1 — Utiliser KEYS * en production

```bash
KEYS *   -- ❌ Parcourt toutes les clés en une seule commande bloquante
```

> [!warning] Bloque tout le serveur pendant son exécution
> Redis étant mono-thread pour l'exécution des commandes (voir [[Redis 02 — Architecture, event loop & threads réseau]]), `KEYS *` sur une base volumineuse bloque toutes les autres commandes pendant toute sa durée. Utiliser `SCAN` (ou `HSCAN`/`SSCAN`/`ZSCAN` selon le type), qui parcourt les clés par petits lots sans bloquer.

---

## 🪤 Piège 2 — Croire que RDB seul suffit à la durabilité

```bash
save 60 1000   -- Une seule stratégie de persistance activée
```

> [!warning] Jusqu'à plusieurs minutes de données perdables
> Entre deux points de sauvegarde RDB, un crash perd toutes les écritures intervenues depuis. Pour une durabilité fine, combiner avec l'AOF (`appendonly yes`, voir [[Redis 08 — Persistance AOF (append-only file)]]) plutôt que de se fier à RDB seul pour des données critiques.

---

## 🪤 Piège 3 — Coder en dur l'adresse du leader avec Sentinel en place

```bash
redis-cli -h 192.168.1.10 -p 6379   -- ❌ Adresse fixe du leader d'hier
```

> [!warning] Le failover de Sentinel devient inefficace
> Si l'application code en dur l'adresse du leader plutôt que d'interroger Sentinel (`SENTINEL get-master-addr-by-name`), un failover automatique ne sert à rien : l'application continue de viser l'ancien leader, potentiellement devenu un simple follower ou hors service. Voir [[Redis 12 — Sentinel (haute disponibilité)]].

---

## 🪤 Piège 4 — Déployer Sentinel ou Galera-like avec un nombre pair de nœuds

```
2 Sentinels, ou 4 nœuds Cluster/Galera — pas de majorité fiable après coupure réseau
```

> [!warning] Toujours un nombre impair, 3 minimum
> Avec un nombre pair de nœuds de vote, une partition réseau peut créer deux groupes strictement égaux, sans majorité déterminable — bloquant le failover ou la validation d'écritures. Voir [[Redis 12 — Sentinel (haute disponibilité)]] et [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]] pour le même principe appliqué à un autre moteur.

---

## 🪤 Piège 5 — Opération multi-clés en Cluster sans hash tag

```bash
MGET utilisateur:42:profil utilisateur:42:preferences
-- (error) CROSSSLOT Keys in request don't hash to the same slot
```

> [!tip] Grouper les clés liées avec un hash tag dès la conception
> `utilisateur:{42}:profil` et `utilisateur:{42}:preferences` partagent le même hash slot grâce à `{42}` — penser au hash tagging dès le nommage des clés si Redis Cluster est envisagé, pas après coup. Voir [[Redis 13 — Cluster (sharding & hash slots)]].

---

## 🪤 Piège 6 — Exposer Redis sans authentification ni pare-feu

```bash
bind 0.0.0.0
# requirepass absent
```

> [!warning] Compromission quasi immédiate si exposé sur Internet
> Une instance Redis sans mot de passe accessible publiquement est une cible automatisée massivement scannée. Toujours combiner `bind` restreint, `requirepass`/ACL, et un pare-feu système — voir [[Redis 14 — Sécurité (ACL, TLS & durcissement)]].

---

## 🪤 Piège 7 — Croire que CONFIG SET persiste au redémarrage

```bash
CONFIG SET maxmemory 2gb
-- Redémarrage du serveur : maxmemory revient à la valeur de redis.conf
```

> [!warning] CONFIG SET ne touche jamais le fichier de configuration
> Un changement via `CONFIG SET` ne vit qu'en mémoire. Sans `CONFIG REWRITE` (ou une modification manuelle de `redis.conf`), le prochain redémarrage annule silencieusement le changement — un piège fréquent après un réglage à chaud fait en urgence puis oublié. Voir [[Redis 15 — Le fichier redis.conf & CONFIG REWRITE]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `KEYS *` en production | Utiliser `SCAN` et ses variantes |
| RDB seul pour des données critiques | Combiner RDB + AOF |
| Adresse du leader codée en dur avec Sentinel | Interroger Sentinel pour l'adresse courante |
| Nombre pair de Sentinels ou de nœuds de vote | Toujours un nombre impair, 3 minimum |
| Opération multi-clés en Cluster sans hash tag | Grouper les clés liées avec `{...}` |
| Instance exposée sans authentification | `requirepass`/ACL + pare-feu systématiques |
| `CONFIG SET` supposé persistant | `CONFIG REWRITE` (ou reporter le changement dans `redis.conf`) |
