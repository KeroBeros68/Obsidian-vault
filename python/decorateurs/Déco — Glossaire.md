#python #decorateurs #glossaire #référence

| Terme | Définition |
|---|---|
| **décorateur** | Fonction qui prend une fonction en entrée et retourne une fonction modifiée |
| **closure** | Fonction qui capture les variables de son environnement extérieur |
| **wrapper** | Fonction interne du décorateur — remplace la fonction originale |
| **@wraps** | Décorateur de `functools` qui copie les métadonnées de la fonction originale sur le wrapper |
| **__wrapped__** | Attribut pointant vers la fonction originale non décorée |
| **fabrique de décorateur** | Fonction qui retourne un décorateur — permet de passer des arguments |
| **decorator factory** | Synonyme de fabrique de décorateur |
| **@property** | Transforme une méthode en attribut calculé avec getter/setter/deleter |
| **@staticmethod** | Méthode sans accès à `self` ni `cls` — simple fonction dans l'espace de la classe |
| **@classmethod** | Méthode avec accès à la classe via `cls` — compatible avec l'héritage |
| **@lru_cache** | Mémoïsation avec cache LRU — `functools.lru_cache` |
| **@cache** | Mémoïsation sans limite — `functools.cache` (Python 3.9+) |
| **@cached_property** | Propriété calculée une fois et mise en cache dans `self.__dict__` |
| **@total_ordering** | Génère les méthodes de comparaison manquantes à partir de `__eq__` et `__lt__` |
| **@contextmanager** | Crée un context manager depuis un générateur |
| **@dataclass** | Génère `__init__`, `__repr__`, `__eq__` automatiquement |
| **@overload** | Déclare des signatures multiples — voir [[Typing 04 — Fonctions — paramètres & retours]] |
| **@runtime_checkable** | Permet `isinstance` sur un Protocol — voir [[Typing 06 — Protocoles & duck typing structurel]] |
| **@field_validator** | Validation Pydantic sur un champ — voir [[Pydantic 06 — @field_validator]] |
| **@model_validator** | Validation Pydantic du modèle entier — voir [[Pydantic 07 — @model_validator]] |
| **@computed_field** | Champ Pydantic calculé — voir [[Pydantic 09 — Alias & computed_field]] |
| **nonlocal** | Mot-clé pour modifier une variable de la closure parente |
| **ParamSpec** | Capture la signature complète d'une fonction — pour typer les décorateurs |
| **TypeVar** | Paramètre de type générique — `R = TypeVar("R")` |
| **empilement** | Appliquer plusieurs décorateurs sur une même fonction |
| **ordre d'application** | Bas → haut : le décorateur le plus proche de la fonction est appliqué en premier |
| **ordre d'exécution** | Haut → bas : le décorateur le plus haut s'exécute en premier |
| **descriptor** | Protocole Python (`__get__`, `__set__`, `__delete__`) — base de `property`, `classmethod`, `staticmethod` |
| **__call__** | Méthode qui rend une instance appelable comme une fonction |
| **mémoïsation** | Mise en cache des résultats d'une fonction pure pour les réutiliser |
| **singleton** | Pattern garantissant qu'une classe n'a qu'une seule instance |
| **registre de plugins** | Dict de fonctions enregistrées via un décorateur |
| **backoff exponentiel** | Stratégie de retry où le délai double à chaque tentative |
