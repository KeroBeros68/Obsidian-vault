#devops #ftp #fondamentaux

## USER / PASS : une authentification en deux commandes

```
USER alice
331 Mot de passe requis pour alice
PASS motdepasse
230 Connexion réussie
```

`USER` déclare l'identité, `PASS` transmet le mot de passe — chacune dans un message texte séparé sur le canal de contrôle.

> [!warning] Aucun chiffrement dans le FTP de base
> Ces deux commandes transitent en clair sur le réseau. Un mot de passe FTP intercepté (analyseur de trafic sur un réseau partagé, point d'accès Wi-Fi non chiffré, proxy intermédiaire compromis) est immédiatement lisible. C'est la raison principale pour laquelle FTP nu ne devrait plus servir à authentifier un accès sensible — voir [[FTP 06 — Sécuriser FTP (FTPS)]] et [[FTP 07 — FTP vs SFTP vs SCP]].

## FTP anonyme : un accès public sans compte dédié

FTP prévoit un mécanisme natif de dépôt public, sans création de compte individuel : le compte conventionnel `anonymous`.

```
USER anonymous
331 Merci de donner votre adresse email comme mot de passe
PASS visiteur@exemple.com
230 Connexion anonyme acceptée
```

Par convention (non technique, jamais vérifiée par le serveur), le mot de passe attendu est une adresse email — un usage hérité de l'époque où cela servait de traçabilité minimale et non contraignante, sans réelle valeur de sécurité.

> [!info] Cas d'usage historique : la distribution de fichiers publics
> Le FTP anonyme a longtemps servi à distribuer des logiciels libres, des miroirs de paquets Linux, ou de la documentation publique — un accès en lecture seule, sans compte à gérer. La plupart de ces usages sont aujourd'hui couverts par HTTP(S) ou des CDN, mais le mécanisme reste disponible dans la quasi-totalité des serveurs FTP.

> [!warning] FTP anonyme ≠ FTP anonyme en écriture
> Un serveur FTP anonyme mal configuré qui autorise aussi l'**upload** (pas seulement la lecture) devient un vecteur d'abus classique : dépôt de fichiers malveillants, exfiltration de données via un répertoire accessible en écriture par n'importe qui. Un FTP anonyme de production doit systématiquement être configuré en lecture seule, sauf besoin métier précis et isolé.

## Comptes nommés et confinement (chroot)

Un serveur FTP en production restreint presque toujours chaque utilisateur authentifié à son propre répertoire (`chroot` — l'utilisateur ne voit et n'accède qu'à un sous-arbre de fichiers défini, comme s'il s'agissait de la racine du système). Ce mécanisme, propre à chaque implémentation serveur, sera détaillé dans les modules dédiés (vsftpd, Pure-FTPd, ProFTPD).

## Pour aller plus loin

Une fois connecté, les commandes de navigation et de transfert (et la distinction entre mode ASCII et binaire) sont couvertes dans [[FTP 04 — Commandes essentielles & modes de transfert]].

Sources : [File Transfer Protocol — Wikipedia](https://en.wikipedia.org/wiki/File_Transfer_Protocol)
