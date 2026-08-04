#bdd #redis #avancé

## MULTI/EXEC : mettre en file d'attente puis exécuter

```bash
MULTI
INCR compteur
SET derniere_maj "2026-07-28"
EXEC
```

Toutes les commandes entre `MULTI` et `EXEC` sont mises en file d'attente côté serveur, puis exécutées **séquentiellement et sans interruption** — aucune autre commande d'un autre client ne peut s'intercaler entre elles, grâce au modèle mono-thread (voir [[Redis 02 — Architecture, event loop & threads réseau]]).

> [!warning] Ce n'est pas un rollback façon SQL
> Si une commande échoue à l'exécution (ex. `INCR` sur une valeur non numérique), les autres commandes de la transaction s'exécutent quand même. Redis ne garantit que l'**isolation** (pas d'entrelacement avec d'autres clients) et l'**atomicité du bloc dans son ensemble face aux autres clients** — pas un rollback automatique en cas d'erreur applicative en cours d'exécution.

## WATCH : le verrouillage optimiste

```bash
WATCH solde:compte42
val = GET solde:compte42
-- calcul côté application --
MULTI
SET solde:compte42 nouvelle_valeur
EXEC   -- retourne nil si solde:compte42 a été modifié entre WATCH et EXEC
```

`WATCH` surveille une ou plusieurs clés : si l'une d'elles est modifiée par un autre client entre le `WATCH` et l'`EXEC`, la transaction entière est annulée (`EXEC` retourne `nil`), et l'application doit réessayer. C'est un modèle *compare-and-swap* plutôt qu'un verrou bloquant.

> [!tip] Le pattern classique lire-modifier-écrire
> `WATCH` + `GET` + calcul applicatif + `MULTI`/`SET`/`EXEC` reproduit un `UPDATE ... WHERE version = ?` optimiste, sans jamais bloquer les autres clients pendant le calcul intermédiaire.

## EVAL : exécuter du Lua côté serveur

```bash
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 cle valeur

EVAL "
local val = redis.call('GET', KEYS[1])
if val == ARGV[1] then
  return redis.call('DEL', KEYS[1])
else
  return 0
end
" 1 verrou:ressource mon_identifiant
```

Un script Lua s'exécute atomiquement du point de vue des autres clients — comme une transaction `MULTI`/`EXEC`, mais avec de la logique conditionnelle et des boucles, impossibles à exprimer avec de simples commandes mises en file d'attente.

> [!info] EVALSHA : éviter de renvoyer le script en entier
> `SCRIPT LOAD` enregistre un script et retourne son hash SHA1 ; `EVALSHA <hash> ...` l'exécute ensuite sans retransmettre le code source à chaque appel — utile pour des scripts fréquemment invoqués depuis l'application.

## Redis Functions : une alternative structurée à EVAL

Depuis Redis 7.0, les **Functions** permettent d'enregistrer des bibliothèques de scripts Lua nommées et versionnées côté serveur, persistées avec le RDB/AOF (contrairement aux scripts `EVAL` bruts, non persistés par défaut) :

```bash
FUNCTION LOAD "#!lua name=mabib
redis.register_function('mafonction', function(keys, args)
  return redis.call('SET', keys[1], args[1])
end)"

FCALL mafonction 1 cle valeur
```

## Pour aller plus loin

Au-delà des transactions, Redis permet de diffuser des messages en temps réel entre clients — voir [[Redis 10 — Pub-Sub & notifications de l'espace de clés]].

Sources : [Redis data types — Redis Documentation](https://redis.io/docs/latest/develop/data-types/), [Redis Open Source 8.0 release notes — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/release-notes/redisce/redisos-8.0-release-notes/)
