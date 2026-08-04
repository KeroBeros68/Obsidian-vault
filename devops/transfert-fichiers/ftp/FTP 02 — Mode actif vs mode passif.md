#devops #ftp #fondamentaux

## Le problème à résoudre : qui ouvre le canal de données ?

Le canal de contrôle (voir [[FTP 01 — Canal de contrôle & canal de données]]) est toujours ouvert par le client vers le serveur, sans ambiguïté. Le canal de données, en revanche, peut être ouvert dans un sens ou dans l'autre — c'est exactement ce que choisissent les modes actif et passif.

## Mode actif : le serveur rappelle le client

```
1. Client → Serveur (contrôle, port 21) : PORT 192,168,1,10,200,15
   (le client annonce : "connecte-toi à moi sur l'IP 192.168.1.10, port 51215")
2. Serveur → Client : ouvre une NOUVELLE connexion depuis son port 20 vers le port annoncé
3. Le transfert a lieu sur cette connexion serveur → client
```

Le client écoute sur un port qu'il choisit, envoie une commande `PORT` pour l'annoncer, et c'est le **serveur** qui initie la connexion de données vers le client.

> [!warning] Le mode actif est structurellement incompatible avec les pare-feu modernes
> Un pare-feu client (ou un routeur NAT) bloque par défaut toute connexion entrante non sollicitée — exactement ce qu'exige le mode actif : une connexion initiée par le serveur *vers* le client. Sans règle explicite ou module `conntrack` FTP dédié côté pare-feu (voir [[FTP 05 — FTP à travers les pare-feu & NAT]]), le mode actif échoue silencieusement dans la quasi-totalité des réseaux domestiques ou d'entreprise actuels.

## Mode passif : le client ouvre les deux connexions

```
1. Client → Serveur (contrôle, port 21) : PASV
2. Serveur → Client : 227 Entering Passive Mode (203,0,113,5,200,15)
   (le serveur annonce : "connecte-toi à moi sur l'IP 203.0.113.5, port 51215")
3. Client → Serveur : ouvre une NOUVELLE connexion vers l'IP/port annoncés
4. Le transfert a lieu sur cette connexion client → serveur
```

En mode passif, c'est le **client** qui initie les deux connexions — le serveur se contente d'écouter sur un port qu'il annonce en retour. Aucune connexion entrante n'est requise côté client.

> [!tip] Pourquoi le mode passif est devenu le standard de fait
> La quasi-totalité des clients FTP modernes (navigateurs, `lftp`, FileZilla) utilisent PASV par défaut, précisément parce que le client se trouve presque toujours derrière un NAT ou un pare-feu personnel qui bloquerait le mode actif. Le mode passif ne pose de problème que côté **serveur** — c'est lui qui doit ouvrir une plage de ports dynamiques à travers son propre pare-feu, un point de configuration classique traité en [[FTP 05 — FTP à travers les pare-feu & NAT]].

## Décoder l'adresse dans une réponse PASV

```
227 Entering Passive Mode (203,0,113,5,200,15)
```

Les six nombres encodent une adresse IPv4 (les quatre premiers : `203.0.113.5`) et un port sur deux octets (les deux derniers : `200 × 256 + 15 = 51215`).

## Récapitulatif

| | Mode actif (PORT) | Mode passif (PASV) |
|---|---------------------|------------------------|
| Qui initie le canal de données | Le serveur, vers le client | Le client, vers le serveur |
| Compatible pare-feu client | Non, sans règle spécifique | Oui, nativement |
| Compatible pare-feu serveur | Oui (le serveur initie sortant) | Nécessite d'ouvrir une plage de ports |
| Usage aujourd'hui | Rare, legacy | Standard de fait |

## Pour aller plus loin

Une fois la mécanique des deux modes comprise, les commandes FTP courantes et les modes de transfert de fichier (ASCII vs binaire) sont couverts dans [[FTP 03 — Authentification & FTP anonyme]].

Sources : [Active Vs. Passive FTP Simplified — JSCAPE](https://www.jscape.com/blog/active-v-s-passive-ftp-simplified), [Active FTP vs. Passive FTP, a Definitive Explanation — slacksite.com](https://slacksite.com/other/ftp.html), [FTP Connection Modes — WinSCP](https://winscp.net/eng/docs/ftp_modes)
