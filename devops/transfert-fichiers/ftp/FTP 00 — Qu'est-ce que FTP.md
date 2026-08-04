#devops #ftp #fondamentaux

## Un protocole plus vieux qu'HTTP

FTP (*File Transfer Protocol*) est normalisé par la RFC 959 en 1985 — près d'une décennie avant HTTP. C'est un protocole client-serveur conçu pour un seul usage : transférer des fichiers entre deux machines, avec des commandes pour naviguer dans une arborescence distante (lister, créer, supprimer, renommer).

> [!info] Toujours largement utilisé, malgré son âge
> FTP reste présent dans de nombreuses infrastructures existantes (hébergement web mutualisé, échange de fichiers avec des partenaires industriels, systèmes embarqués) — moins par choix technique aujourd'hui que par inertie et compatibilité historique. Comprendre FTP reste utile même si un nouveau projet ne devrait, dans l'immense majorité des cas, pas le choisir en 2026.

## Ce que FTP n'est pas

> [!warning] FTP n'est pas chiffré par défaut
> Dans sa forme originelle (celle décrite dans ce module de base), FTP transmet identifiants et données en clair sur le réseau — n'importe qui en position d'observer le trafic peut lire un mot de passe FTP en clair. Les variantes sécurisées (FTPS, ou son remplaçant le plus courant aujourd'hui SFTP) sont couvertes respectivement en [[FTP 06 — Sécuriser FTP (FTPS)]] et [[FTP 07 — FTP vs SFTP vs SCP]].

## Une particularité structurelle : deux connexions, pas une

Contrairement à HTTP ou SSH, qui font transiter commandes et données sur une seule connexion TCP, FTP en utilise **deux, séparées** : un canal de contrôle pour les commandes et les réponses, un canal de données pour le contenu des fichiers eux-mêmes. Cette séparation, unique parmi les protocoles applicatifs courants, est la source de la plupart des complications pratiques de FTP (pare-feu, NAT) — détaillée en [[FTP 01 — Canal de contrôle & canal de données]].

## À quoi ressemble une session FTP

```
$ ftp ftp.exemple.com
Connected to ftp.exemple.com.
220 Bienvenue sur le serveur FTP
Name: alice
331 Mot de passe requis
Password: ********
230 Connexion réussie
ftp> ls
ftp> get rapport.pdf
ftp> quit
```

## Pour aller plus loin

Le mécanisme des deux canaux — et pourquoi il complique tant les choses avec les pare-feu modernes — est détaillé dans [[FTP 01 — Canal de contrôle & canal de données]].

Sources : [File Transfer Protocol — Wikipedia](https://en.wikipedia.org/wiki/File_Transfer_Protocol), [File Transfer Protocol - FTP — firewall.cx](https://www.firewall.cx/networking/network-protocols/protocols-ftp.html)
