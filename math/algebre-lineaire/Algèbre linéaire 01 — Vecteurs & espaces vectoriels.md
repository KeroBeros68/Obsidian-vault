#math #algebre-lineaire #vecteurs

## Vecteur

Un vecteur dans ℝⁿ est une liste ordonnée de n nombres réels.

```
      ⎡ v₁ ⎤
v =   ⎢ v₂ ⎥   ∈ ℝⁿ
      ⎣ v₃ ⎦
```

En pratique : un point dans l'espace (direction + magnitude), ou un jeu de données (features d'un exemple).

## Opérations de base

**Addition** — élément par élément :
```
⎡1⎤   ⎡4⎤   ⎡5⎤
⎢2⎥ + ⎢5⎥ = ⎢7⎥
⎣3⎦   ⎣6⎦   ⎣9⎦
```

**Multiplication scalaire** :
```
    ⎡2⎤   ⎡ 6⎤
3 × ⎢1⎥ = ⎢ 3⎥
    ⎣4⎦   ⎣12⎦
```

## Norme (longueur)

La norme euclidienne (L2) mesure la longueur du vecteur :

```
‖v‖ = √(v₁² + v₂² + ... + vₙ²)
```

```
‖[3, 4]‖ = √(9 + 16) = √25 = 5
```

Autres normes :
```
‖v‖₁ = |v₁| + |v₂| + ... + |vₙ|          (norme L1)
‖v‖∞ = max(|v₁|, |v₂|, ..., |vₙ|)        (norme infinie)
```

**Vecteur unitaire** (normalisation) :
```
v̂ = v / ‖v‖   →   ‖v̂‖ = 1
```

## Produit scalaire (dot product)

```
u · v = u₁v₁ + u₂v₂ + ... + uₙvₙ
```

```
[1,2,3] · [4,5,6] = 1×4 + 2×5 + 3×6 = 4 + 10 + 18 = 32
```

**Interprétation géométrique** :
```
u · v = ‖u‖ × ‖v‖ × cos(θ)
```

| cos(θ) | Angle θ | Interprétation |
|--------|---------|----------------|
| 1 | 0° | Vecteurs parallèles, même sens |
| 0 | 90° | Vecteurs orthogonaux (perpendiculaires) |
| -1 | 180° | Vecteurs antiparallèles |

## Produit vectoriel (cross product) — ℝ³ uniquement

```
u × v = ⎡u₂v₃ - u₃v₂⎤
        ⎢u₃v₁ - u₁v₃⎥
        ⎣u₁v₂ - u₂v₁⎦
```

Résultat : vecteur perpendiculaire au plan défini par u et v.
Magnitude : ‖u × v‖ = ‖u‖ × ‖v‖ × sin(θ) = aire du parallélogramme.

## Espace vectoriel

Un espace vectoriel est un ensemble de vecteurs fermé sous l'addition et la multiplication scalaire.

**Propriétés clés :**
- Vecteur nul : u + 0 = u
- Commutativité : u + v = v + u
- Distributivité : α(u + v) = αu + αv

**Sous-espace** : sous-ensemble fermé sous les mêmes opérations, contenant le vecteur nul.

## Base et dimension

Une **base** est un ensemble de vecteurs **linéairement indépendants** qui **engendrent** l'espace.

```
Base standard de ℝ³ :
e₁ = [1,0,0]    e₂ = [0,1,0]    e₃ = [0,0,1]

Tout vecteur v = [a,b,c] = a×e₁ + b×e₂ + c×e₃
```

La **dimension** est le nombre de vecteurs dans une base.

> [!tip] Indépendance linéaire
> Des vecteurs sont linéairement indépendants si aucun n'est combinaison linéaire des autres.
> Test : α₁v₁ + α₂v₂ + ... = 0 implique α₁ = α₂ = ... = 0.
