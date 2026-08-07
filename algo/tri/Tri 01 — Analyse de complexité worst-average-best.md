#algo #tri #complexité #bases #algoritmo #ordenación #complejidad #fundamentos

## Análisis de complejidad:  worst / average / best peor / medio / mejor

La complejidad de un algoritmo de ordenación se mide en **número de comparaciones** (u operaciones elementales) en función del tamaño n de la matriz.

## Los tres casos que hay que distinguir

| Caso | Significado | Ejemplo concreto |
|-----|---------------|-----------------|
| **Mejor caso** | Entrada más favorable posible | Matriz ya ordenada |
| **Caso medio** | Entrada aleatoria (esperanza) | Permutación uniforme |
| **Peor caso** | Entrada más desfavorable posible | Matriz ordenada al revés |

## Tabla comparativa general

| Algoritmo | Mejor | Promedio | Peor | Espacio | Estable |
|------------|------|---------|-------|--------|--------|
| **Ordenación por fusión** | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ |
| **Ordenación rápida** | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ |
| **Ordenación por montones** | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ |
| **Ordenación por recuento** | O(n + k) | O(n + k) | O(n + k) | O(k) | ✅ |
| **Ordenación por radix** | O(d·(n+k)) | O(d·(n+k)) | O(d·(n+k)) | O(n+k) | ✅ |
| Ordenación por inserción | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Ordenación por selección | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Ordenación por burbujas | O(n) | O(n²) | O(n²) | O(1) | ✅ |

*k = rango de valores, d = número de dígitos (base)*

## El límite inferior Ω(n log n)

Ningún algoritmo de ordenación **por comparaciones** puede superar O(n log n) en el caso medio.

```
Demostración mediante árbol de decisión:
- n elementos → n! permutaciones posibles
- Cada comparación binaria divide los casos en dos
- Altura mínima del árbol = ⌈log₂(n!)⌉
- Según la aproximación de Stirling: log₂(n!) ≈ n log₂ n
→ Se necesitan al menos n log n comparaciones. ✅
```

Las ordenaciones por recuento y por radix **no se ajustan a este límite**, ya que no comparan los elementos entre sí.

## Estabilidad — definición

Una ordenación es **estable** si dos elementos de igual valor conservan su orden relativo original.

```python
data = [(“Bob”, 2), (“Alice”, 1), (“Charlie”, 2)]

# Tras una ordenación estable por valor:
# [(“Alice”, 1), (“Bob”, 2), (“Charlie”, 2)]
#                 ↑                ↑
#           Bob antes que Charlie — se respeta el orden original ✅

# Tras una ordenación inestable:
# [(“Alice”, 1), (“Charlie”, 2), (“Bob”, 2)]  ← puede ocurrir ❌
```

## Ordenación in situ frente a ordenación externa

```
In situ: O(1) de espacio auxiliar. Modifica la lista original.
            → Ordenación rápida, ordenación por montones, ordenación por inserción.

Externa: O(n) o más espacio auxiliar. Crea matrices temporales.
            → Ordenación por fusión (O(n)), ordenación por recuento (O(k)), ordenación por radix (O(n+k)).

Externa estable + O(n log n) en el peor caso: ordenación por fusión — la opción por defecto en la práctica.
```

## Cuándo utilizar cada tipo de ordenación

| Situación | Ordenación recomendada | Motivo |
|-----------|---------------|--------|
| Uso general | Ordenación rápida (introsort) | Rápida en la práctica, espacio O(log n) |
| Se requiere estabilidad | Ordenación por fusión | Estable + O(n log n) garantizado |
| Memoria limitada | Ordenación por montones | Espacio O(1), O(n log n) garantizado |

| Enteros pequeños acotados | Ordenación por recuento | O(n + k) si k es razonable |
| Enteros grandes / cadenas | Ordenación por radix | O(d·n) independiente del número de comparaciones |
| Matriz casi ordenada | Ordenación por inserción | O(n) sobre datos casi ordenados |

> [!tip] Regla práctica
> En competiciones o entrevistas: ordenación por fusión si se requiere estabilidad; ordenación rápida en caso contrario. En producción: utiliza la ordenación nativa del lenguaje (`sorted()` en Python = Timsort, híbrido de fusión e inserción).

> [!info] Timsort
> Python, Java y Android utilizan **Timsort**, un híbrido entre ordenación por fusión y por inserción que detecta las subsecuencias ya ordenadas. Complejidad: O(n) en el mejor caso, O(n log n) en el peor, estable.
