#python #tqdm #glossaire #référence

| Terme | Définition |
|---|---|
| **tqdm** | Librairie Python de barres de progression — "taqaddum" (progrès en arabe) |
| **tqdm.auto** | Import auto-détectant l'environnement (CLI vs Jupyter) |
| **tqdm.notebook** | Backend Jupyter — rendu widget HTML interactif |
| **tqdm.asyncio** | Backend asyncio — compatible avec `as_completed` et `gather` |
| **tqdm.rich** | Backend rich — barres stylisées avec couleurs |
| **trange** | Raccourci pour `tqdm(range(...))` |
| **pbar.update(n)** | Avancer la barre de n unités |
| **pbar.set_description()** | Changer le label à gauche de la barre |
| **pbar.set_postfix()** | Afficher un dict de métriques à droite de la barre |
| **pbar.set_postfix_str()** | Afficher une chaîne libre à droite de la barre |
| **pbar.write()** | Afficher un message sans casser la barre |
| **pbar.reset()** | Remettre la barre à zéro pour réutilisation |
| **pbar.clear()** | Effacer temporairement l'affichage de la barre |
| **pbar.close()** | Fermer et libérer la barre proprement |
| **desc** | Paramètre — label descriptif affiché à gauche |
| **total** | Paramètre — nombre total d'itérations (obligatoire pour les générateurs) |
| **unit** | Paramètre — unité affichée (défaut : "it") |
| **unit_scale** | Paramètre — active l'affichage 1k / 1M / 1G |
| **unit_divisor** | Paramètre — 1000 (défaut) ou 1024 (octets) |
| **leave** | Paramètre — `True` = barre reste après fin, `False` = disparaît |
| **position** | Paramètre — ligne d'affichage pour les barres imbriquées |
| **disable** | Paramètre — `True` = aucun affichage (mode silencieux) |
| **ncols** | Paramètre — largeur fixe de la barre en caractères |
| **dynamic_ncols** | Paramètre — adapte automatiquement la largeur au terminal |
| **bar_format** | Paramètre — format d'affichage entièrement personnalisable |
| **initial** | Paramètre — valeur de départ non-zéro (reprise de progression) |
| **smoothing** | Paramètre — lissage de la vitesse (0 = moyenne, 1 = instantané) |
| **colour** | Paramètre — couleur de la barre (nom ou code hex) |
| **postfix** | Zone d'affichage de métriques à droite de la barre |
| **it/s** | Itérations par seconde — métrique de vitesse de la barre |
| **s/it** | Secondes par itération — affiché quand < 1 it/s |
| **progress_apply** | Méthode Pandas activée par `tqdm.pandas()` |
| **thread_map** | `tqdm.contrib.concurrent` — threading avec barre automatique |
| **process_map** | `tqdm.contrib.concurrent` — multiprocessing avec barre automatique |
| **tenumerate** | `tqdm.contrib` — tqdm + enumerate |
| **tzip** | `tqdm.contrib` — tqdm + zip |
