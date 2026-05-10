#c #memoire #valgrind #asan #debugging #outils

## Catégories d'erreurs mémoire

| Erreur | Description |
|--------|-------------|
| **Memory leak** | Mémoire allouée jamais libérée |
| **Use-after-free** | Accès à mémoire libérée |
| **Double free** | `free` appelé deux fois sur le même pointeur |
| **Heap overflow** | Écriture au-delà d'un bloc alloué |
| **Stack overflow** | Dépassement de la pile |
| **Uninitialized read** | Lecture de mémoire non initialisée |

## Valgrind — détection à l'exécution

```bash
# Compiler avec symboles de debug
gcc -g -O0 prog.c -o prog

# Lancer sous valgrind
valgrind --leak-check=full --track-origins=yes ./prog
```

### Sortie typique

```
==1234== Invalid read of size 4
==1234==    at 0x4005B3: main (prog.c:12)
==1234==  Address 0x5204040 is 0 bytes after a block of size 40 alloc'd

==1234== LEAK SUMMARY:
==1234==    definitely lost: 40 bytes in 1 blocks
==1234==    indirectly lost: 0 bytes in 0 blocks
```

### Options utiles

```bash
--leak-check=full          # afficher chaque fuite en détail
--track-origins=yes        # tracer l'origine des valeurs non initialisées
--show-leak-kinds=all      # inclure les fuites indirectes et possibles
--error-exitcode=1         # retourner 1 si erreurs → utile en CI
```

> [!info] Valgrind ralentit de ×10 à ×50
> À utiliser pour le debug, pas pour les benchmarks. Ne pas compiler avec `-O2` : les optimisations masquent des erreurs.

## AddressSanitizer (ASan) — intégré au compilateur

Plus rapide que Valgrind (×2 environ). Détecte les erreurs à la compilation et à l'exécution.

```bash
gcc -g -fsanitize=address -fsanitize=undefined prog.c -o prog
./prog
```

### Sortie typique

```
==1234==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x...
READ of size 4 at 0x... thread T0
    #0 0x4005b3 in main prog.c:12
```

### Autres sanitizers

```bash
-fsanitize=address      # heap/stack overflow, use-after-free, double-free
-fsanitize=undefined    # UB : division par 0, overflow entier signé, etc.
-fsanitize=thread       # race conditions (TSan)
-fsanitize=memory       # lectures non initialisées (MSan — clang seulement)
```

## Comparaison Valgrind vs ASan

| | Valgrind | ASan |
|--|---------|------|
| Overhead | ×10 à ×50 | ×2 |
| Compilation spéciale | Non | Oui (`-fsanitize=address`) |
| Fuites mémoire | ✅ | Partiel (`-fsanitize=leak`) |
| Use-after-free | ✅ | ✅ |
| Valeurs non initialisées | ✅ | Non (MSan) |
| Stack overflow | Partiel | ✅ |
| Race conditions | Non | TSan séparé |

## Workflow recommandé

```bash
# 1. Développement : ASan toujours activé
gcc -g -fsanitize=address,undefined -o prog prog.c && ./prog

# 2. Debug de fuite ou d'unitialisé : Valgrind
valgrind --leak-check=full --track-origins=yes ./prog

# 3. Race conditions : ThreadSanitizer
gcc -g -fsanitize=thread -o prog prog.c && ./prog
```

> [!tip] Ne jamais combiner ASan et Valgrind
> Ils instrumentent tous les deux le heap et entrent en conflit. Choisir l'un ou l'autre.
