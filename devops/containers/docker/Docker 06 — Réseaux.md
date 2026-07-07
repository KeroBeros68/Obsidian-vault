#devops #docker #réseaux

## Les drivers réseau

Docker attache chaque conteneur à un réseau virtuel via un **driver**, choisi selon le besoin d'isolation, de performance, ou de portée (un seul hôte ou plusieurs).

| Driver | Portée | Cas d'usage |
|--------|--------|-------------|
| **bridge** (défaut) | Un seul hôte | Communication entre conteneurs sur la même machine |
| **host** | Un seul hôte | Performance maximale, pas d'isolation réseau |
| **none** | Aucune | Isolation totale (tâches sans accès réseau) |
| **overlay** | Plusieurs hôtes | Swarm, conteneurs répartis sur plusieurs machines |
| **macvlan** | Un seul hôte | Attribue une adresse MAC/IP virtuelle au conteneur, visible directement sur le réseau physique |
| **ipvlan** | Un seul hôte | Comme macvlan, mais MAC partagée avec l'hôte — supporte le routage L3 |

Approfondissement de overlay, macvlan, ipvlan et des limites précises du mode host dans [[Docker 12 — Réseaux avancés (Overlay, Macvlan, Ipvlan)]].

## Bridge par défaut vs bridge personnalisé

```bash
# Bridge par défaut — communication uniquement par IP
docker run -d --name web nginx
docker run -d --name app myapp
# 'app' ne peut PAS résoudre 'web' par son nom

# Bridge personnalisé — résolution DNS automatique par nom
docker network create my-net
docker run -d --network my-net --name web nginx
docker run -d --network my-net --name app myapp
# 'app' peut faire ping vers 'web' directement
```

Le bridge par défaut isole les conteneurs des autres réseaux et de l'extérieur, mais ne fournit **pas** de résolution DNS par nom — seul un réseau bridge créé explicitement (`docker network create`) l'offre.

```bash
# Bridge avec un plan d'adressage explicite
docker network create --driver bridge --subnet 10.10.0.0/24 --gateway 10.10.0.1 mon-reseau

# Empêcher les conteneurs de ce réseau de communiquer entre eux (isolation stricte)
docker network create --opt com.docker.network.bridge.enable_icc=false reseau-isole
```

> [!info] `enable_icc=false` par réseau vs `icc: false` global
> L'option `--opt com.docker.network.bridge.enable_icc=false` désactive la communication inter-conteneurs sur **ce réseau précis** ; le réglage `icc: false` de `daemon.json` (voir [[Docker 10 — Configuration production & nettoyage]]) l'applique par défaut à tout nouveau bridge. Les deux agissent au même niveau, l'un ciblé, l'autre global.

## Configurer un réseau dans Docker Compose

Compose crée automatiquement un réseau bridge personnalisé par projet (voir [[Docker 07 — Docker Compose]]), mais chaque réseau déclaré sous la clé top-level `networks:` peut être configuré finement plutôt que laissé par défaut.

```yaml
networks:
  frontend:
    driver: bridge

  backend:
    driver: bridge
    internal: true            # aucune route sortante — isolé du reste du monde
    ipam:
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1
```

| Clé | Rôle |
|-----|------|
| `driver` | Driver réseau à utiliser (`bridge`, `overlay`...) — voir la table des drivers plus haut |
| `driver_opts` | Options spécifiques au driver (ex. options du bridge Linux natif) |
| `ipam.config` | Plan d'adressage explicite : `subnet`, `gateway`, `ip_range` — sinon Docker choisit automatiquement |
| `internal` | `true` : aucun accès depuis/vers l'extérieur du réseau (pas de route par défaut) |
| `attachable` | Autorise `docker network connect` à attacher manuellement un conteneur hors Compose à ce réseau |
| `name` | Fixe le nom réel du réseau plutôt que de le laisser préfixé par le nom du projet |
| `external` | Réutilise un réseau déjà créé ailleurs plutôt que d'en créer un nouveau |

## Réseau externe et adressage par service

```yaml
networks:
  shared:
    external: true
    name: my-preexisting-net   # réseau déjà créé via `docker network create`, partagé entre projets
```

```yaml
services:
  db:
    image: postgres:17
    networks:
      backend:
        ipv4_address: 172.28.0.10   # IP statique — nécessite un subnet fixe déclaré via ipam
        aliases:
          - database                 # nom DNS additionnel, en plus du nom du service
```

## Cas particuliers

> [!warning] ipv4_address nécessite un subnet fixe déclaré
> Assigner une IP statique à un service exige que le réseau utilise un plan d'adressage explicite (`ipam.config` avec un `subnet`) — impossible sur un réseau dont Docker choisit l'adressage automatiquement.

> [!tip] internal: true pour isoler un réseau backend
> Un réseau marqué `internal: true` n'a aucune route vers l'extérieur : les conteneurs qui y sont connectés ne peuvent pas atteindre Internet. Utile pour une base de données qui ne doit dialoguer qu'avec les services applicatifs, jamais sortir directement.

> [!info] Nom de réseau généré vs explicite
> Par défaut, Compose préfixe chaque réseau du nom du projet (ex. `monprojet_frontend`, visible via `docker network ls`). La clé `name:` permet de fixer un nom exact — utile pour un réseau partagé entre plusieurs fichiers Compose ou référencé par un outil externe.

## EXPOSE vs publication de port

```dockerfile
EXPOSE 3000   # documentation uniquement — ne publie rien
```

```bash
# Seule cette commande publie réellement le port vers l'hôte
docker run -p 8080:3000 myapp
```

`EXPOSE` dans un Dockerfile sert uniquement de documentation pour les autres développeurs et certains outils — il ne rend le port accessible nulle part. Seul `-p` (ou `ports:` en Compose) ouvre réellement un accès depuis l'hôte.

```bash
# ❌ Accessible depuis n'importe quelle interface réseau de l'hôte, y compris publique
docker run -p 0.0.0.0:3306:3306 mysql

# ✅ Accessible uniquement depuis l'hôte lui-même
docker run -p 127.0.0.1:3306:3306 mysql
```

> [!warning] `-p 3306:3306` publie sur `0.0.0.0` par défaut
> Sans IP explicite, `-p` écoute sur toutes les interfaces de l'hôte — y compris une interface publique si la machine en a une. Pour un service qui ne doit être atteint que localement (base de données consultée uniquement par un autre conteneur ou par l'hôte), préfixer par `127.0.0.1:` limite l'exposition.

## Déconnecter un conteneur et nettoyer les réseaux

```bash
docker network ls                              # lister les réseaux
docker network disconnect mon-reseau mon-app   # déconnecter sans arrêter le conteneur
docker network rm mon-reseau                   # supprimer un réseau précis (doit être inutilisé)
docker network prune                           # supprimer tous les réseaux non utilisés
```

`docker network disconnect` isole un conteneur d'un réseau donné sans l'arrêter ni le supprimer — utile pour retirer temporairement un service d'un réseau backend pendant une maintenance, par exemple.

## Checklist de debug réseau

Quand un conteneur ne communique pas avec un autre ou avec l'extérieur, vérifier dans cet ordre :

```bash
# 1. Le port est-il bien mappé côté hôte ?
docker port mon-conteneur
# Résultat attendu : 8080/tcp -> 0.0.0.0:8080
curl http://localhost:8080

# 2. Le conteneur est-il bien sur le réseau attendu ?
docker network inspect mon-reseau   # chercher le conteneur dans "Containers"

# 3. Les deux conteneurs se joignent-ils l'un l'autre ?
docker exec autre-conteneur ping mon-conteneur
docker exec autre-conteneur curl http://mon-conteneur:8080

# 4. La résolution DNS interne fonctionne-t-elle ?
docker exec mon-conteneur cat /etc/resolv.conf
docker exec mon-conteneur nslookup autre-conteneur

# 5. L'application écoute-t-elle sur la bonne interface ?
docker exec mon-conteneur ss -tlnp
# L'app doit écouter sur 0.0.0.0:PORT, pas 127.0.0.1:PORT
```

| Symptôme | Cause probable | Solution |
|----------|------------------|----------|
| Timeout depuis l'hôte | Port mapping absent ou incorrect | Ajouter `-p 8080:80` |
| Timeout entre conteneurs | Conteneurs pas sur le même réseau | Connecter les deux au même `docker network` |
| "Connection refused" | L'application écoute sur `127.0.0.1` au lieu de `0.0.0.0` **dans le conteneur** | Configurer l'application pour écouter sur `0.0.0.0` |
| Conteneur ne joint jamais Internet | NAT/masquerade désactivé côté hôte | Vérifier `sysctl net.ipv4.ip_forward` (doit valoir `1`) |

```bash
# Voir les règles NAT que Docker a injectées dans iptables
sudo iptables -L -n -t nat | grep -i docker

# Logs du démon Docker filtrés sur le réseau (utile si network create/connect échoue)
journalctl -u docker.service | grep -i network
```

> [!info] Une image sans `ping`/`curl`/`netstat` ?
> Beaucoup d'images minimales (slim, alpine, distroless) n'embarquent aucun outil réseau. Lancer un conteneur "boîte à outils" sur le même réseau contourne le problème sans modifier l'image : `docker run -it --rm --network container:mon-conteneur nicolaka/netshoot` donne accès à `ping`, `curl`, `dig`, `tcpdump`.

## Cas particuliers

> [!warning] host = perte d'isolation
> Le mode `--network host` fait partager au conteneur la pile réseau complète de l'hôte (mêmes ports, mêmes interfaces). Gain de performance réel, mais surface d'attaque élargie — à réserver à des cas précis (ex. outils de monitoring du host).

> [!tip] Toujours un bridge personnalisé
> Pour des conteneurs qui doivent se parler par nom, créer systématiquement un réseau bridge personnalisé plutôt que de compter sur le bridge par défaut.

> [!info] DNS interne
> Sur un réseau bridge personnalisé, Docker fournit un serveur DNS interne (`127.0.0.11`) qui résout automatiquement les noms de conteneurs.
