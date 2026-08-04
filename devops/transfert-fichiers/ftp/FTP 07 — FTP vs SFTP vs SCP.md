#devops #ftp #sécurité #avancé

## La confusion la plus fréquente : SFTP n'est pas « FTP sécurisé »

Le nom prête à confusion, mais **SFTP** (*SSH File Transfer Protocol*) n'a aucun lien de parenté technique avec FTP. C'est un protocole entièrement différent, conçu dès le départ comme une extension du protocole SSH — il ne réutilise ni les commandes, ni l'architecture à deux canaux, ni aucun mécanisme de FTP (voir [[FTP 01 — Canal de contrôle & canal de données]]).

> [!warning] FTPS et SFTP ne sont pas deux variantes du même protocole
> FTPS (voir [[FTP 06 — Sécuriser FTP (FTPS)]]) est littéralement FTP avec du TLS ajouté par-dessus — même protocole, canal chiffré. SFTP est un protocole SSH à part entière, qui ne parle pas FTP du tout. Confondre les deux mène à des erreurs de configuration (pointer un client SFTP vers un port FTPS, ou l'inverse) qui échouent immédiatement.

## SFTP : un seul canal, hérité de SSH

```
Client ──── connexion SSH unique (port 22) ────► Serveur
              (authentification + commandes + données, tout dans le même tunnel chiffré)
```

SFTP profite de tout ce que SSH offre déjà : chiffrement de bout en bout, authentification par clé publique ou mot de passe, un seul port à ouvrir (22, le même que pour une connexion shell classique). Il n'y a ni canal de données séparé, ni mode actif/passif, ni négociation TLS distincte — toute la complexité réseau propre à FTP (voir [[FTP 02 — Mode actif vs mode passif]] et [[FTP 05 — FTP à travers les pare-feu & NAT]]) disparaît simplement parce que le modèle à deux connexions n'existe pas dans SSH.

## SCP : plus simple encore, mais plus limité

**SCP** (*Secure Copy Protocol*), lui aussi basé sur SSH, ne fait qu'une seule chose : copier un fichier d'un point A à un point B. Pas de navigation interactive, pas de reprise de transfert interrompu, pas de listing de répertoire structuré — une commande, un transfert.

```bash
scp rapport.pdf utilisateur@serveur:/home/utilisateur/documents/
```

> [!info] SCP est aujourd'hui considéré obsolète par OpenSSH lui-même
> Le projet OpenSSH recommande depuis plusieurs années d'utiliser SFTP plutôt que SCP, y compris pour de simples copies ponctuelles — le protocole SCP historique a des faiblesses de validation d'arguments corrigées différemment selon les implémentations. `scp` reste disponible et fonctionnel, mais `sftp` (ou un client SFTP graphique) est la recommandation actuelle par défaut.

## Comparatif d'ensemble

| | FTP | FTPS | SFTP | SCP |
|---|-----|------|------|-----|
| Chiffrement | Aucun | TLS (ajouté à FTP) | SSH (natif) | SSH (natif) |
| Nombre de canaux | 2 (contrôle + données) | 2 (tous deux à sécuriser séparément) | 1 | 1 |
| Port(s) | 21 + dynamique | 21 ou 990 + dynamique | 22 (unique) | 22 (unique) |
| Compatible pare-feu/NAT sans configuration spécifique | Non | Non | Oui | Oui |
| Navigation interactive (listing, renommage...) | Oui | Oui | Oui | Non |
| Authentification par clé publique | Non (nativement) | Possible (certificat client TLS) | Oui (héritée de SSH) | Oui (héritée de SSH) |
| Statut en 2026 | Legacy, à éviter pour du sensible | Toujours utilisé dans certains secteurs réglementés | Standard de facto pour un nouveau projet | Fonctionnel mais déconseillé par OpenSSH |

> [!tip] Pour un nouveau projet aujourd'hui : SFTP par défaut
> Sauf contrainte héritée (partenaire industriel n'acceptant que du FTPS, logiciel legacy ne parlant que FTP), SFTP est le choix par défaut raisonnable : un seul port à gérer, pas de mode actif/passif à trancher, authentification par clé déjà standard dans tout environnement qui utilise SSH.

## Pour aller plus loin

Ceci conclut le module FTP de base — voir [[FTP — Index des fiches]] pour une vue d'ensemble. Les implémentations serveur concrètes (vsftpd, Pure-FTPd, ProFTPD), qui appliquent ces concepts à des configurations réelles, feront l'objet de modules dédiés ultérieurs.

Sources : [SFTP vs. FTPS: What are the key differences? — SolarWinds](https://www.solarwinds.com/resources/it-glossary/sftp-vs-ftps), [FTP vs SFTP vs FTPS: Differences, Security, Ports, Use Cases — sftptogo.com](https://sftptogo.com/blog/what-is-the-difference-between-ftp-sftp-and-ftps/), [SFTP, Secure FTP, FTP/SSL, FTPS, FTP, SCP... What's the difference? — sftp.net](https://www.sftp.net/sftp-vs-ftps)
