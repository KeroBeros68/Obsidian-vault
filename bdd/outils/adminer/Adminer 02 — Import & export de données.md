#bdd #adminer #fondamentaux #import #export

## Exporter depuis Adminer

L'onglet "Exporter" d'une base ou d'une table propose :

| Option | Détail |
|--------|--------|
| Portée | Base entière, tables sélectionnées, ou résultat d'une requête SQL |
| Format | SQL (`CREATE` + `INSERT`), CSV, CSV avec en-têtes |
| Structure | Avec ou sans `DROP TABLE` préalable, avec ou sans données |
| Compression | Aucune, gzip, zip (selon build PHP disponible) |
| Sortie | Téléchargement direct ou affichage dans le navigateur |

```sql
-- Extrait typique d'un export SQL Adminer
DROP TABLE IF EXISTS `clients`;
CREATE TABLE `clients` (
  `id` int NOT NULL AUTO_INCREMENT,
  `nom` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;

INSERT INTO `clients` (`id`, `nom`) VALUES (1, 'Alice Martin');
```

> [!tip] Export d'un résultat de requête personnalisé
> Exécuter d'abord la requête dans l'onglet SQL, puis utiliser le lien d'export associé au résultat — permet d'extraire un sous-ensemble filtré sans écrire de clause d'export manuelle.

## Importer dans Adminer

L'onglet "Importer" accepte :

- un fichier `.sql` (ou `.sql.gz`, `.sql.zip` selon configuration PHP)
- du SQL collé directement dans un champ texte
- l'exécution immédiate ou requête par requête, avec arrêt à la première erreur

> [!warning] Limites PHP à vérifier avant un gros import
> `upload_max_filesize` et `post_max_size` (php.ini) plafonnent la taille du fichier importable via le formulaire web. Pour une base volumineuse, l'import direct en ligne de commande (`mysql < dump.sql`, `psql -f dump.sql`) reste plus fiable et plus rapide qu'un import via l'interface web.

## CSV : cas particulier

L'export CSV ne conserve pas la structure (types, contraintes, index) — seulement les données. Pour un aller-retour fidèle entre deux instances du même moteur, préférer l'export SQL complet ; réserver le CSV aux échanges avec des outils tiers (tableur, script Python, autre SGBD).

## Pour aller plus loin

Sur un moteur donné, les outils natifs (`mysqldump`/MySQL Shell pour MySQL, `pg_dump` pour PostgreSQL) restent la référence pour des sauvegardes de production — voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]]. Adminer reste adapté aux exports ponctuels et aux migrations manuelles de faible volume, pas à une stratégie de sauvegarde planifiée.

Sources : [Adminer — site officiel](https://www.adminer.org/en/), [Adminer sur GitHub](https://github.com/vrana/adminer/)
