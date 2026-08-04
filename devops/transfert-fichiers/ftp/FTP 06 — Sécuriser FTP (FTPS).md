#devops #ftp #sécurité #avancé

## FTPS : FTP avec TLS, pas un nouveau protocole

FTPS (*FTP Secure*, parfois appelé FTP over TLS/SSL) ajoute une couche de chiffrement TLS au protocole FTP existant — les mêmes commandes, les mêmes deux canaux (voir [[FTP 01 — Canal de contrôle & canal de données]]), mais chiffrés. Ce n'est pas un protocole différent, contrairement à SFTP (voir [[FTP 07 — FTP vs SFTP vs SCP]]).

## Deux façons d'établir le TLS : implicite et explicite

**FTPS implicite** : le chiffrement TLS est obligatoire dès la connexion, sur un port dédié distinct du FTP nu (traditionnellement 990 pour le contrôle).

```
Client se connecte sur le port 990 → poignée de main TLS immédiate → puis USER/PASS chiffrés
```

**FTPS explicite** (la forme recommandée aujourd'hui) : le client se connecte d'abord en clair sur le port FTP standard (21), puis demande explicitement l'activation du TLS avant de s'authentifier.

```
Client → Serveur (port 21, en clair) : AUTH TLS
Serveur → Client : 234 Prêt pour la négociation TLS
--- poignée de main TLS ---
Client → Serveur (désormais chiffré) : USER alice
```

| | FTPS implicite | FTPS explicite |
|---|-------------------|---------------------|
| Port de contrôle | Dédié (990 par convention) | Standard (21) |
| Négociation TLS | Automatique dès la connexion | Demandée via `AUTH TLS` |
| Compatibilité avec un client FTP non sécurisé | Aucune (rejeté d'emblée) | Le serveur peut refuser ou accepter une connexion en clair selon sa politique |
| Recommandation | Historique, moins flexible | Généralement préférée aujourd'hui |

> [!warning] Le canal de données doit aussi être protégé explicitement
> Sécuriser le canal de contrôle ne suffit pas : sans la commande `PBSZ 0` puis `PROT P` (*Protection Level: Private*), le canal de données peut rester non chiffré même après un `AUTH TLS` réussi sur le contrôle — un fichier transféré en clair malgré une authentification protégée. Vérifier explicitement que `PROT P` est actif avant tout transfert sensible.

## Le vrai problème pratique de FTPS : deux canaux à sécuriser, pas un

> [!warning] FTPS complique le passage des pare-feu plus encore que FTP nu
> Puisque le canal de contrôle est chiffré, un pare-feu ne peut plus inspecter les commandes `PASV`/`PORT` pour deviner le port de données (contrairement au mécanisme `nf_conntrack_ftp` couvert en [[FTP 05 — FTP à travers les pare-feu & NAT]], qui a besoin de lire le contenu en clair). FTPS derrière un pare-feu impose donc presque toujours de fixer une plage de ports passifs explicite, sans pouvoir compter sur un helper de suivi de connexion.

## Certificats : les mêmes règles que pour n'importe quel service TLS

Un serveur FTPS a besoin d'un certificat (auto-signé pour un usage interne, signé par une autorité reconnue pour un accès public) — les mêmes principes que pour un serveur web (voir [[MySQL 24 — Chiffrement TLS]] pour un exemple détaillé de configuration TLS transposable dans ses grandes lignes).

## Pour aller plus loin

FTPS n'est qu'une des deux réponses courantes au problème du FTP non chiffré — la clarification complète face à SFTP (un protocole entièrement différent) se trouve dans [[FTP 07 — FTP vs SFTP vs SCP]].

Sources : [FTPS Vs. SFTP: Is FTPS Still Relevant in 2026? — sftptogo.com](https://sftptogo.com/blog/is-ftps-still-relevant/), [Understanding Key Differences Between FTP, FTPS And SFTP — JSCAPE](https://www.jscape.com/blog/understanding-key-differences-between-ftp-ftps-and-sftp)
