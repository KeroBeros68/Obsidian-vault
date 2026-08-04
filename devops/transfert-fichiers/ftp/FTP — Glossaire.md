#devops #ftp #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **RFC 959** | Norme de 1985 définissant le protocole FTP — toujours la référence de base, malgré de nombreuses extensions ultérieures. |
| **Canal de contrôle** | Connexion TCP persistante (port 21) transportant les commandes et codes de réponse d'une session FTP. |
| **Canal de données** | Connexion TCP éphémère, ouverte pour chaque transfert de fichier ou listing, distincte du canal de contrôle. |
| **Mode actif (PORT)** | Mode où le serveur initie la connexion de données vers un port annoncé par le client — incompatible avec la plupart des pare-feu/NAT modernes. |
| **Mode passif (PASV)** | Mode où le client initie les deux connexions (contrôle et données) — standard de fait aujourd'hui. |
| **FTP anonyme** | Mécanisme d'accès public sans compte dédié, via l'identifiant conventionnel `anonymous`. |
| **`TYPE A` / `TYPE I`** | Modes de transfert ASCII (traduit les fins de ligne) et Image/binaire (octets bruts, sans transformation). |
| **`nf_conntrack_ftp`** | Module noyau Linux inspectant le canal de contrôle FTP pour autoriser dynamiquement les connexions de données à travers un pare-feu. |
| **Plage de ports passifs** | Intervalle de ports restreint configuré côté serveur pour ses connexions de données, permettant une règle de pare-feu praticable. |
| **`masquerade_address`** | Adresse IP publique qu'un serveur FTP derrière NAT doit annoncer dans ses réponses `PASV`, à la place de son IP privée. |
| **FTPS (FTP Secure)** | FTP classique avec chiffrement TLS ajouté — mêmes commandes, mêmes deux canaux, canal chiffré. |
| **FTPS implicite / explicite** | Implicite : TLS obligatoire dès la connexion (port dédié 990). Explicite : connexion en clair puis activation TLS via `AUTH TLS`. |
| **`PROT P`** | Commande FTPS activant explicitement le chiffrement du canal de données (*Protection Level: Private*), distinct du chiffrement du canal de contrôle. |
| **SFTP (SSH File Transfer Protocol)** | Protocole de transfert de fichiers bâti sur SSH, sans lien technique avec FTP malgré le nom — un seul canal chiffré, pas de mode actif/passif. |
| **SCP (Secure Copy Protocol)** | Protocole de copie de fichier simple basé sur SSH, sans navigation interactive — aujourd'hui déconseillé par OpenSSH au profit de SFTP. |
| **Chroot (FTP)** | Confinement d'un utilisateur FTP authentifié à un sous-arbre de fichiers défini, l'empêchant de voir le reste du système de fichiers. |
