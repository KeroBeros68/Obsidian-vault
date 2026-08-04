#devops #pure-ftpd #avancé

## PassivePortRange : restreindre les ports de données

```bash
echo "49152 49252" | sudo tee /etc/pure-ftpd/conf/PassivePortRange
sudo systemctl restart pure-ftpd
```

Sans cette option, Pure-FTPd choisit un port de données passif n'importe où dans la plage éphémère du système — trop large pour être ouverte sans risque sur un pare-feu. Restreindre à une plage connue permet une règle de pare-feu praticable, comme détaillé de façon générique en [[FTP 05 — FTP à travers les pare-feu & NAT]].

```bash
sudo iptables -A INPUT -p tcp --dport 49152:49252 -j ACCEPT
```

## ForcePassiveIP : l'équivalent Pure-FTPd du masquerade_address

```bash
echo "203.0.113.10" | sudo tee /etc/pure-ftpd/conf/ForcePassiveIP
sudo systemctl restart pure-ftpd
```

Quand le serveur est lui-même derrière un NAT, `ForcePassiveIP` force Pure-FTPd à annoncer cette adresse publique dans ses réponses `PASV`, plutôt que sa propre adresse IP privée — sans quoi un client externe reçoit une IP interne injoignable (voir le piège générique déjà documenté en [[FTP — Pièges classiques]]).

> [!warning] ForcePassiveIP fige l'IP annoncée pour toutes les connexions
> Une fois cette option définie, Pure-FTPd annonce systématiquement cette adresse, y compris pour des clients qui se connecteraient depuis le réseau local (où l'IP privée réelle serait plus appropriée). Sur un serveur accessible à la fois en interne et depuis Internet, ce compromis mérite d'être évalué au cas par cas — certaines distributions proposent des mécanismes de détection conditionnelle, absents du cœur de Pure-FTPd.

## Le cas du conteneur ou de la VM avec redirection de port

```bash
# Hôte : redirige le port 21 et la plage passive vers le conteneur
# docker run -p 21:21 -p 49152-49252:49152-49252 ...
```

> [!tip] La plage passive doit être redirigée en entier, pas seulement le port 21
> Une redirection de port qui n'ouvre que le port de contrôle (21) laisse le canal de données inatteignable — `PassivePortRange` et `ForcePassiveIP` doivent être configurés de pair avec une redirection couvrant l'intégralité de la plage annoncée, sans quoi la connexion s'établit mais tout transfert échoue (le symptôme caractéristique déjà couvert en [[FTP — Pièges classiques]]).

## Vérifier le comportement réel

```bash
ftp -p serveur   # -p force le mode passif côté client
ftp> ls
227 Entering Passive Mode (203,0,113,10,192,20)
```

Décoder la réponse (voir [[FTP 02 — Mode actif vs mode passif]]) confirme que l'IP et le port annoncés correspondent bien à ce qui est joignable depuis l'extérieur.

## Pour aller plus loin

Une fois le service fonctionnel et accessible, une checklist de durcissement rassemble les points de sécurité couverts dans les fiches précédentes — voir [[Pure-FTPd 08 — Sécurité & durcissement]].

Sources : [How to Set Up Pure-FTPd to Bind to a Specific IPv4 Address — oneuptime.com](https://oneuptime.com/blog/post/2026-03-20-pure-ftpd-bind-specific-ipv4/view), [Pure FTP Passive Mode Issue — kb.unixservertech.com](https://kb.unixservertech.com/software/pure-ftpd/passive-mode)
