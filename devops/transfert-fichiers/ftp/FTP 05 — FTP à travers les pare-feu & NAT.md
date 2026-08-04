#devops #ftp #avancé

## Le problème concret : un port qu'on ne connaît pas à l'avance

Un pare-feu classique n'autorise que les connexions vers des ports connus à l'avance (21 pour le contrôle FTP). Or le canal de données (voir [[FTP 02 — Mode actif vs mode passif]]) s'ouvre sur un port **choisi dynamiquement** au moment de la session, annoncé en clair dans une réponse `PASV` — un pare-feu qui ne comprend que TCP/IP n'a aucun moyen de savoir que ce port doit être autorisé.

## Solution 1 : un module de suivi de connexion dédié à FTP

Sous Linux, le module noyau `nf_conntrack_ftp` inspecte le contenu du canal de contrôle, repère les commandes `PASV`/`PORT` et le port qu'elles négocient, puis autorise dynamiquement la connexion de données correspondante — sans qu'aucune règle de pare-feu explicite par port ne soit nécessaire.

```bash
sudo modprobe nf_conntrack_ftp

# nftables/iptables modernes : associer le helper à la connexion de contrôle
sudo iptables -t raw -A PREROUTING -p tcp --dport 21 -j CT --helper ftp

# Autoriser les connexions RELATED détectées par le helper
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

> [!info] RELATED : la connexion de données est rattachée à la connexion de contrôle
> Une fois le helper actif, le pare-feu classe la connexion de données comme `RELATED` (liée à une connexion de contrôle FTP déjà établie) plutôt que comme une connexion entrante non sollicitée — c'est ce mécanisme qui permet d'accepter le trafic FTP sans ouvrir une large plage de ports en permanence.

## Solution 2 : fixer une plage de ports passifs

Si le module de suivi n'est pas disponible ou pas souhaité (certaines architectures réseau complexes, load balancers), l'alternative consiste à configurer le serveur FTP pour n'utiliser qu'une plage de ports limitée et connue pour ses connexions passives, puis à n'ouvrir que cette plage sur le pare-feu :

```
# Configuration serveur FTP (exemple générique, syntaxe propre à chaque implémentation)
passive_ports_min = 49152
passive_ports_max = 49252
```

```bash
sudo iptables -A INPUT -p tcp --dport 49152:49252 -j ACCEPT
```

> [!tip] Une plage réduite plutôt que tous les ports éphémères
> Sans configuration explicite, un serveur FTP choisit un port passif n'importe où dans la plage éphémère du système (souvent 32768-60999 sous Linux) — bien trop large à ouvrir sans risque sur un pare-feu. Restreindre la plage à quelques centaines de ports (comme ci-dessus) rend la règle de pare-feu praticable, au prix d'un nombre de connexions simultanées plafonné par la taille de la plage.

## Le cas particulier du NAT côté serveur

Quand le serveur FTP lui-même est derrière un NAT (hébergé sur un réseau privé, exposé via une redirection de port), la réponse `PASV` doit annoncer l'**adresse publique**, pas l'adresse privée du serveur — sans quoi le client tente de se connecter à une IP interne injoignable depuis l'extérieur.

```
# Configuration serveur FTP (syntaxe générique)
masquerade_address = 203.0.113.5   # IP publique à annoncer dans les réponses PASV
```

> [!warning] Un symptôme caractéristique : le transfert de fichier échoue, la connexion réussit
> Si l'authentification et la navigation (`LIST`) fonctionnent mais que tout transfert (`RETR`/`STOR`) reste bloqué ou expire, la cause la plus fréquente est une adresse IP privée renvoyée dans la réponse `PASV` d'un serveur derrière NAT — un piège classique couvert plus en détail dans [[FTP — Pièges classiques]].

## Pour aller plus loin

Une fois le fonctionnement réseau de FTP maîtrisé, sa sécurisation via TLS (FTPS) est couverte dans [[FTP 06 — Sécuriser FTP (FTPS)]].

Sources : [How to Use ip_conntrack_ftp for Passive FTP Through iptables — oneuptime.com](https://oneuptime.com/blog/post/2026-03-20-iptables-ftp-passive-conntrack/view), [Passive Mode FTP with iptables — reliablepenguin.com](https://blogs.reliablepenguin.com/2012/03/08/passive-mode-ftp-with-iptables)
