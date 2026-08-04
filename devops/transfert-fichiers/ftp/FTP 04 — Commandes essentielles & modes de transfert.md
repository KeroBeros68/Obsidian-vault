#devops #ftp #fondamentaux

## Naviguer et transférer

| Commande | Rôle |
|----------|------|
| `PWD` | Affiche le répertoire courant sur le serveur |
| `CWD <dossier>` | Change de répertoire (*Change Working Directory*) |
| `LIST` | Liste le contenu du répertoire, format détaillé (façon `ls -l`) |
| `NLST` | Liste le contenu, format simple (noms seuls) |
| `RETR <fichier>` | Télécharge un fichier depuis le serveur |
| `STOR <fichier>` | Envoie un fichier vers le serveur |
| `DELE <fichier>` | Supprime un fichier distant |
| `MKD` / `RMD` | Crée / supprime un répertoire distant |
| `RNFR` / `RNTO` | Renomme un fichier (*Rename From* / *Rename To*, en deux commandes) |
| `QUIT` | Ferme la session proprement |

```
CWD /public/documents
250 Répertoire changé avec succès
RETR rapport.pdf
150 Ouverture de la connexion de données
226 Transfert terminé
```

> [!info] `RNFR`/`RNTO` : deux commandes pour une seule opération
> Renommer un fichier demande deux échanges successifs — `RNFR ancien_nom` (« je veux renommer ce fichier ») puis `RNTO nouveau_nom` (« vers ce nom »). Une déconnexion entre les deux laisse l'opération inachevée sans erreur explicite.

## TYPE : ASCII vs binaire, une distinction héritée

```
TYPE A   # ASCII : traduit les fins de ligne selon le système d'exploitation
TYPE I   # Image (binaire) : transfert brut, octet pour octet
```

FTP distingue deux modes de transfert : `TYPE A` (ASCII) convertit les fins de ligne (`\r\n` vs `\n`) entre systèmes différents lors du transfert de fichiers texte, tandis que `TYPE I` (*Image*, en pratique « binaire ») transmet les octets tels quels, sans aucune transformation.

> [!warning] Le mauvais TYPE corrompt silencieusement les fichiers binaires
> Transférer une image, un exécutable ou une archive en mode `TYPE A` peut altérer son contenu si le serveur ou le client réinterprète certains octets comme des fins de ligne à convertir. La plupart des clients modernes basculent automatiquement en binaire pour les extensions non textuelles, mais un script FTP manuel doit explicitement envoyer `TYPE I` avant tout transfert de fichier non-texte.

## SYST et FEAT : découvrir les capacités du serveur

```
SYST
215 UNIX Type: L8

FEAT
211-Extensions supportées :
 MDTM
 SIZE
 UTF8
211 End
```

`SYST` révèle le système d'exploitation sous-jacent (information parfois masquée pour des raisons de sécurité), `FEAT` liste les extensions au-delà du socle RFC 959 — utile pour savoir si un serveur supporte par exemple `SIZE` (taille exacte d'un fichier avant transfert) ou `MDTM` (date de dernière modification).

## Pour aller plus loin

Une fois la mécanique de base acquise, le vrai casse-tête pratique de FTP — le faire fonctionner à travers un pare-feu ou une passerelle NAT — est détaillé dans [[FTP 05 — FTP à travers les pare-feu & NAT]].

Sources : [File Transfer Protocol — Wikipedia](https://en.wikipedia.org/wiki/File_Transfer_Protocol)
