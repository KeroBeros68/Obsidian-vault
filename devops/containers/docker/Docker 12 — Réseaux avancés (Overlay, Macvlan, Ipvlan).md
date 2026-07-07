#devops #docker #réseaux #avancé

## Au-delà du bridge

[[Docker 06 — Réseaux]] couvre le quotidien (bridge, publication de port, debug). Trois drivers répondent à des besoins plus spécifiques : communication multi-hôtes (overlay), intégration au réseau physique existant (macvlan/ipvlan), ou absence totale de réseau (none) — ainsi que les limites concrètes du mode host.

## Overlay : réseau multi-hôtes

Le driver **overlay** encapsule le trafic conteneur dans des paquets UDP (**VXLAN**) pour faire communiquer des conteneurs situés sur des machines physiques différentes — l'équivalent d'un VPN privé entre les hôtes Docker.

Prérequis : un **Swarm** initialisé (le cas courant), un cluster Kubernetes avec le plugin CNI approprié, ou Docker standalone avec un key-value store externe (etcd, Consul).

```bash
# 1. Initialiser Swarm (si pas déjà fait)
docker swarm init

# 2. Créer le réseau overlay — --attachable autorise des conteneurs
#    standalone (hors service Swarm) à le rejoindre
docker network create --driver overlay --attachable mon-overlay

# 3. Déployer des services dessus
docker service create --name api --network mon-overlay --replicas 3 mon-api:latest
docker service create --name db --network mon-overlay postgres:15

# Vérifier la communication inter-nœuds
docker exec $(docker ps -q -f name=api) ping -c 3 db
```

| Option | Rôle |
|--------|------|
| `--attachable` | Autorise des conteneurs standalone à rejoindre le réseau |
| `--encrypted` | Chiffre le trafic overlay via IPsec |
| `--subnet` / `--gateway` | Plan d'adressage du réseau overlay |

> [!warning] Le trafic overlay n'est pas chiffré par défaut
> Sans `--opt encrypted`, les données transitent en clair entre les hôtes. Pour des données sensibles, activer le chiffrement (`docker network create --driver overlay --opt encrypted ...`) — au prix d'une surcharge CPU d'environ 10-15 % et d'une latence légèrement accrue. À réserver au trafic qui en a réellement besoin.

## Macvlan : approfondissement

Chaque conteneur reçoit sa **propre adresse MAC** et apparaît comme un périphérique physique distinct sur le réseau local (voir le driver déjà listé dans [[Docker 06 — Réseaux]]).

| Mode | Comportement |
|------|--------------|
| `bridge` (défaut) | Les conteneurs communiquent entre eux et avec le réseau externe |
| `private` | Les conteneurs sont isolés les uns des autres |
| `passthru` | Un seul conteneur par interface physique, accès direct |
| `VEPA` | Le trafic repasse par un switch externe capable de *hairpin* |

```bash
docker network create --driver macvlan \
  --subnet 192.168.1.0/24 --gateway 192.168.1.1 \
  -o parent=eth0 \
  mon-macvlan

docker run -d --network mon-macvlan --ip 192.168.1.100 --name serveur-web nginx
# serveur-web est accessible en 192.168.1.100 depuis tout le réseau local
```

> [!warning] Le conteneur ne peut pas joindre l'hôte directement
> C'est une limitation intrinsèque du driver macvlan : un conteneur sur ce réseau ne peut pas communiquer avec la machine hôte elle-même. Si cette communication est nécessaire, il faut créer une interface macvlan secondaire dédiée sur l'hôte.

## Ipvlan : alternative à macvlan

Contrairement à macvlan, tous les conteneurs **partagent l'adresse MAC de l'hôte** — seules les adresses IP sont distinctes.

| Critère | Macvlan | Ipvlan |
|---------|---------|--------|
| Adresse MAC | Unique par conteneur | Partagée (celle de l'hôte) |
| Restriction MAC du switch/hyperviseur | Peut poser problème | Aucune |
| Mode L3 (routage) | Non supporté | ✅ Supporté |

```bash
# Mode L2 : conteneurs sur le même segment que l'hôte (comme macvlan)
docker network create --driver ipvlan -o ipvlan_mode=l2 -o parent=eth0 \
  --subnet 192.168.1.0/24 ipvlan-l2

# Mode L3 : routage entre sous-réseaux, sans broadcast ARP
docker network create --driver ipvlan -o ipvlan_mode=l3 -o parent=eth0 \
  --subnet 10.10.0.0/24 ipvlan-l3
```

> [!tip] Quand préférer ipvlan à macvlan
> Environnement où le switch ou l'hyperviseur limite le nombre d'adresses MAC par port, forte densité de conteneurs (évite l'explosion du nombre de MAC), ou besoin explicite de routage niveau IP (mode L3).

## None : ce qui reste possible sans réseau

`--network none` ne laisse que l'interface loopback (`127.0.0.1`) — aucune interface réseau réelle n'est attachée.

```bash
docker run --network none -d --name batch-job mon-script
docker exec batch-job ip addr        # seule 'lo' apparaît
docker exec batch-job ping -c 1 google.com   # Network is unreachable
```

Un conteneur en mode `none` reste joignable par d'autres moyens : **volumes partagés** (échange de fichiers avec l'hôte), **signaux Unix** (`docker kill -s SIGNAL`), **stdin/stdout** (`docker attach`, `docker exec`). Utile pour des jobs batch, des tests de sécurité, ou l'exécution de code non fiable en sandbox.

## Host : les limitations concrètes

Au-delà de la perte d'isolation déjà signalée dans [[Docker 06 — Réseaux]] :

- **Pas de port mapping** : `-p` n'a aucun effet, le conteneur utilise directement les ports de l'hôte.
- **Conflits de ports** : si deux conteneurs en mode host utilisent le même port, le second échoue au démarrage.
- **Non disponible sur Docker Desktop** (macOS/Windows) : le mode host ne fonctionne que sur un hôte Linux natif, puisque Docker Desktop fait déjà tourner le démon dans une VM.

## Tableau récapitulatif des 6 drivers

| Driver | Isolation | Multi-hôtes | Cas d'usage principal |
|--------|-----------|--------------|------------------------|
| bridge | Bonne | ❌ | Développement, apps multi-conteneurs sur un hôte |
| host | Aucune | ❌ | Performance critique, monitoring |
| overlay | Bonne | ✅ | Production distribuée, Swarm |
| macvlan | Excellente | ❌ | Applications legacy, intégration réseau physique |
| ipvlan | Excellente | ❌ | Restrictions MAC, haute densité, routage L3 |
| none | Totale | ❌ | Jobs batch, sandbox, tests de sécurité |

## Cas particuliers

> [!tip] Un réseau par application, pas un seul réseau global
> Créer un bridge (ou overlay) dédié par application ou par stack (`frontend-net`, `backend-net`, `monitoring-net`) isole les incidents et simplifie le debugging, plutôt qu'un unique réseau partagé par tout l'hôte.

> [!info] MTU et overlay
> Une latence inhabituelle sur un réseau overlay vaut la peine de vérifier la MTU effective — l'encapsulation VXLAN réduit la taille utile des paquets, ce qui peut provoquer de la fragmentation si le MTU n'est pas ajusté en conséquence.
