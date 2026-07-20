#linux #home #index

> [!info] Domaine en construction
> Squelette préparé le 2026-07-07 à partir de la table des matières de [Admin-Serveurs — blog.stephane-robert.info](https://blog.stephane-robert.info/docs/admin-serveurs/). D'autres sources viendront enrichir ou réviser ce plan. Aucune fiche n'est encore rédigée — les liens ci-dessous sont volontairement des liens morts qui seront comblés module par module.

## Sous-catégories prévues

### Opérations & SRE (`linux/operations/`)
- [[Operations — Index des fiches]] ← ✅ complet : réduction du travail ingrat (toil), dette technique, patch management, baseline & drift, gestion des capacités

### Fondamentaux (`linux/fondamentaux/`)
- [[Linux — Découverte & Arborescence — Index des fiches]] ← distributions, terminal/SSH, FHS, navigation, chemins
- [[Linux — Fichiers & Texte — Index des fiches]] ← lecture, recherche, grep/sed/awk (bases), pipes, tar/gzip, nano/vi
- [[Linux — Utilisateurs, Droits & Processus — Index des fiches]] ← users/groupes, permissions rwx, sudo, PID/signaux
- [[Linux — Paquets & Réseau — Index des fiches]] ← apt/dnf/apk, mises à jour, bases réseau, systemd (intro), journalctl, dépannage élémentaire

### Exploitation avancée (`linux/exploitation/`)
- [[Linux — Édition & Texte avancé — Index des fiches]] ← Vim, sed/awk avancé, regex, cut/tr/sort/uniq/diff, strace
- [[Linux — Transfert & Archivage — Index des fiches]] ← tar avancé, curl/wget, scp/sftp, rsync, checksums
- [[SSH — Index des fiches]] ← clés, config client/serveur, sshd_config, exécution distante, tunnels (-L/-R/-D)
- [[Linux — Processus & Sessions avancées — Index des fiches]] ← ps/top/htop, nice, jobs bg/fg, nohup
- [[Linux — Observabilité système — Index des fiches]] ← free/vmstat, perf CPU/disque/réseau, identification système
- *(Automatisation Bash et planification cron : déjà couverts par [[Bash — Index des fiches]] et [[Cron — Index des fiches]] — pas de doublon prévu)*

### Maintenance & production (`linux/maintenance/`)
- [[Linux — Gestion de paquets multi-distro — Index des fiches]] ← apt/dnf/apk/pacman/zypper, fpm, dépendances cassées
- [[Systemd — Index des fiches]] ← units, targets, dépendances, dépannage service
- [[Linux — Santé système — Index des fiches]] ← mémoire/swap/OOM, charge (load average), zombies, espace disque
- [[Linux — Stockage & Systèmes de fichiers — Index des fiches]] ← partitions, ext4/XFS, fstab, LVM, RAID (mdadm), LUKS, quotas, NFS/SMB/iSCSI
- [[Linux — Journaux & Rotation — Index des fiches]] ← journalctl approfondi, logrotate
- [[Linux — Réseau & Accès — Index des fiches]] ← Netplan, NetworkManager

### Sécurisation & durcissement (`linux/securite/`)
- [[Linux — Audit & Conformité — Index des fiches]] ← CIS Benchmarks, ANSSI BP-28, OpenSCAP, Lynis
- [[Linux — Durcissement Comptes & Privilèges — Index des fiches]] ← gestion utilisateurs, durcissement sudo
- [[Linux — Permissions avancées & ACL — Index des fiches]] ← ACL, noexec/nosuid/nodev
- [[Firewall — Index des fiches]] ← stateful/stateless, DMZ, firewalld, UFW
- [[Linux — Noyau & Démarrage sécurisé — Index des fiches]] ← sysctl, KSPP, modules noyau, GRUB/Secure Boot
- [[Linux — Sécuriser les services — Index des fiches]] ← systemd sandboxing (PrivateTmp, NoNewPrivileges...)
- [[Linux — Audit système (auditd) — Index des fiches]]
- [[SELinux & AppArmor — Index des fiches]] ← MAC, contextes vs profiles, fapolicyd
- *(SSH — durcissement : rattaché au module [[SSH — Index des fiches]] plutôt que dupliqué ici)*

### Dépannage (`linux/depannage/`)
- [[Linux — Méthodologie de dépannage — Index des fiches]] ← approche systématique, logs d'abord, dépannage réseau/stockage/perf/démarrage

## Hors périmètre / déjà couvert ailleurs

- **Internals (cgroups, namespaces, capabilities)** → déjà traités dans [[Conteneurs — Généralités]]
- **Certifications (RHCSA, LFCS)** → non retenu comme contenu de vault (hors périmètre PKM)
- **Automatisation Bash avancée, shells alternatifs (zsh/fish), planification (cron/at/systemd timers)** → voir [[Bash — Index des fiches]] et [[Cron — Index des fiches]]

## Prérequis & suite

- [[Home]] ← retour accueil
- [[DevOps — Home]] ← domaine complémentaire (conteneurs, nginx, secrets applicatifs)
- [[Manques]] ← autres modules manquants du vault, hors périmètre Linux
