#devops #ftp #pièges #erreurs #debugging

## 🪤 Piège 1 — Le listing fonctionne, le transfert de fichier bloque

```
LIST   → fonctionne
RETR fichier.pdf → bloque indéfiniment ou expire
```

> [!warning] Le symptôme classique d'un mode actif bloqué par un pare-feu
> Si la connexion et la navigation fonctionnent mais que tout transfert reste bloqué, le mode actif tente probablement d'ouvrir une connexion entrante que le pare-feu client refuse silencieusement. Basculer en mode passif (`PASV`, activé par défaut dans la plupart des clients modernes) résout la majorité de ces cas — voir [[FTP 02 — Mode actif vs mode passif]].

---

## 🪤 Piège 2 — Un serveur FTP derrière NAT annonce son IP privée

```
227 Entering Passive Mode (192,168,1,50,200,15)
-- Le client tente de se connecter à 192.168.1.50, injoignable depuis l'extérieur
```

> [!warning] `masquerade_address` doit pointer vers l'IP publique
> Sans configuration explicite, un serveur derrière NAT annonce sa propre IP privée dans ses réponses `PASV` — inutilisable pour un client externe. Configurer `masquerade_address` (ou l'équivalent selon l'implémentation serveur) avec l'IP publique réelle. Voir [[FTP 05 — FTP à travers les pare-feu & NAT]].

---

## 🪤 Piège 3 — Transférer un binaire en mode ASCII

```
TYPE A
STOR image.jpg
-- Fichier corrompu à l'arrivée
```

> [!warning] TYPE A convertit les fins de ligne, y compris dans un binaire
> Un fichier non textuel transféré en `TYPE A` peut voir certains octets réinterprétés comme des fins de ligne à convertir, corrompant silencieusement son contenu. Toujours vérifier ou forcer `TYPE I` avant le transfert d'une image, d'une archive ou d'un exécutable — voir [[FTP 04 — Commandes essentielles & modes de transfert]].

---

## 🪤 Piège 4 — Autoriser l'écriture sur un compte FTP anonyme

```
anonymous → accès en lecture ET écriture sur /incoming/
```

> [!warning] Un vecteur d'abus classique
> Un répertoire anonyme accessible en écriture devient rapidement un dépôt de fichiers malveillants ou un relais d'exfiltration. Un accès anonyme de production doit rester strictement en lecture seule, sauf besoin métier précis, isolé, et surveillé. Voir [[FTP 03 — Authentification & FTP anonyme]].

---

## 🪤 Piège 5 — Confondre le port FTPS implicite et le port FTP standard

```bash
# ❌ Client configuré pour FTPS implicite (990) contre un serveur qui écoute en FTP standard (21)
ftp --ssl-implicit serveur.exemple.com:21
```

> [!warning] FTPS implicite et explicite n'utilisent pas forcément le même port
> FTPS implicite attend une négociation TLS dès l'ouverture de la connexion, sur un port dédié (990 par convention) — s'y connecter en supposant du FTP en clair (port 21) échoue immédiatement. Vérifier quel mode (implicite/explicite) le serveur attend avant de configurer le client. Voir [[FTP 06 — Sécuriser FTP (FTPS)]].

---

## 🪤 Piège 6 — Chercher un « mot de passe FTP sécurisé » au lieu de changer de protocole

> [!tip] Le vrai problème n'est pas le mot de passe, c'est le protocole
> Aucune complexité de mot de passe ne protège contre une transmission en clair sur le réseau. Face à un besoin de sécurité, la question n'est pas « comment sécuriser ce mot de passe FTP » mais « faut-il plutôt utiliser FTPS ou SFTP » — voir [[FTP 07 — FTP vs SFTP vs SCP]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Transfert bloqué malgré une connexion/listing qui fonctionne | Basculer en mode passif |
| Serveur NAT annonçant une IP privée en PASV | Configurer `masquerade_address` avec l'IP publique |
| Binaire transféré en `TYPE A` | Forcer `TYPE I` avant tout fichier non-texte |
| Accès anonyme en écriture | Restreindre en lecture seule par défaut |
| Confusion port FTPS implicite (990) / standard (21) | Vérifier le mode attendu par le serveur |
| Recherche d'un « FTP sécurisé » par mot de passe fort | Passer à FTPS ou SFTP plutôt que renforcer le mot de passe |
