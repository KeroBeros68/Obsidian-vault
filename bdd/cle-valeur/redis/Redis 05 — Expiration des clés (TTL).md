#bdd #redis #fondamentaux

## EXPIRE et le TTL

N'importe quelle clé Redis, quel que soit son type, peut recevoir une durée de vie (*Time To Live*) au-delà de laquelle elle disparaît automatiquement.

```bash
SET session:42 "actif"
EXPIRE session:42 3600        # Expire dans 1 heure
TTL session:42                # Temps restant en secondes (-1 = pas d'expiration, -2 = clé absente)

SETEX cache:page 60 "html"    # Équivalent à SET + EXPIRE en une commande
PEXPIRE cle 5000               # Expiration en millisecondes
EXPIREAT cle 1785000000        # Expiration à un timestamp Unix précis
PERSIST session:42             # Retire l'expiration, la clé redevient permanente
```

> [!warning] Une commande d'écriture sur une clé retire son TTL par défaut
> `SET cle nouvelle_valeur` sans option réinitialise le TTL existant — la clé redevient permanente. Utiliser `SET cle valeur KEEPTTL` pour conserver l'expiration en cours lors d'une mise à jour de valeur.

## Comment Redis expire réellement les clés

Redis ne vérifie pas en permanence l'horloge de chaque clé. Deux mécanismes complémentaires assurent l'expiration :

1. **Expiration passive** : à chaque accès à une clé (`GET`, `HGET`...), Redis vérifie d'abord si son TTL est dépassé — si oui, elle est supprimée avant même de répondre à la commande.
2. **Expiration active** : un cycle périodique en arrière-plan échantillonne aléatoirement un lot de clés avec TTL et supprime celles qui sont expirées, évitant qu'une clé jamais relue reste en mémoire indéfiniment.

> [!info] Pourquoi ce compromis plutôt qu'un minuteur par clé
> Un minuteur précis par clé coûterait cher à grande échelle (potentiellement des millions de clés à TTL). L'échantillonnage périodique garantit une purge en temps borné sans le coût d'une structure de timers par clé.

## Keyspace notifications : réagir à l'expiration

Redis peut publier un événement Pub/Sub (voir [[Redis 10 — Pub-Sub & notifications de l'espace de clés]]) à chaque expiration ou suppression de clé :

```bash
CONFIG SET notify-keyspace-events Ex   # E = keyevent, x = expired

SUBSCRIBE __keyevent@0__:expired
```

> [!tip] Cas d'usage : verrou distribué avec libération automatique
> Un `SET verrou:ressource identifiant NX EX 30` pose un verrou qui s'auto-libère après 30 secondes même si le processus qui le détient crashe — combiné à une notification d'expiration, une autre instance peut réagir immédiatement à la libération du verrou plutôt que d'attendre un TTL fixe.

## Pour aller plus loin

Quand la mémoire disponible est saturée avant même que les TTL n'expirent naturellement, une politique d'éviction prend le relais — détaillée dans [[Redis 06 — Politiques d'éviction (maxmemory-policy)]].

Sources : [Redis data types — Redis Documentation](https://redis.io/docs/latest/develop/data-types/)
