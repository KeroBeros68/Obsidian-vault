#c #pthread #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **thread** | Fil d'exécution au sein d'un processus. Partage le code, le heap et les données globales avec les autres threads du même processus. |
| **pthread_t** | Type opaque identifiant un thread. Ne pas comparer avec `==` — utiliser `pthread_equal`. |
| **section critique** | Portion de code accédant à une ressource partagée. Ne doit être exécutée que par un seul thread à la fois. |
| **race condition** | Bug où le résultat dépend de l'ordre d'exécution non déterministe des threads. |
| **mutex** | *Mutual exclusion.* Verrou permettant à un seul thread d'entrer en section critique. |
| **deadlock** | Blocage circulaire : deux threads ou plus s'attendent mutuellement indéfiniment. |
| **livelock** | Les threads s'exécutent mais ne progressent pas — chacun réagit à l'autre sans avancer. |
| **starvation** | Un thread n'obtient jamais le CPU ou un verrou car d'autres sont toujours prioritaires. |
| **variable de condition** | Mécanisme permettant à un thread de dormir en attendant qu'une condition soit vraie. Toujours associée à un mutex. |
| **spurious wakeup** | Réveil d'un thread depuis `pthread_cond_wait` sans que `signal` ait été appelé. Garanti possible par POSIX → toujours réévaluer la condition avec `while`. |
| **thread joinable** | Thread dont les ressources sont conservées après sa fin jusqu'à un `pthread_join`. État par défaut. |
| **thread detaché** | Thread qui libère ses ressources automatiquement à sa fin. `pthread_join` impossible. |
| **thread zombie** | Thread terminé mais non joiné et non detaché. Ses ressources restent allouées. |
| **TLS** | *Thread-Local Storage.* Variable dont chaque thread possède sa propre instance. `__thread` ou `pthread_key_t`. |
| **spinlock** | Verrou où le thread attend en boucle active au lieu de dormir. Efficace pour très courtes sections critiques sur multi-CPU. |
| **barrier** | Point de synchronisation où N threads s'attendent mutuellement avant de continuer. |
| **point d'annulation** | Appel système ou fonction POSIX où un thread peut être annulé si `pthread_cancel` a été appelé. |
| **thread pool** | Ensemble fixe de threads réutilisables qui exécutent des tâches depuis une file. Évite le coût de création/destruction répétée. |
| **producteur/consommateur** | Pattern où des threads produisent des données dans une file et d'autres les consomment. |
