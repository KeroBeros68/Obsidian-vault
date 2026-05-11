#python #oop #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **classe** | Modèle (blueprint) définissant attributs et méthodes. Créée avec `class`. |
| **instance** | Objet concret créé depuis une classe. Chaque instance a ses propres attributs. |
| **`self`** | Référence à l'instance courante. Premier paramètre de toute méthode d'instance. |
| **`__init__`** | Méthode d'initialisation appelée automatiquement à la création d'une instance. |
| **attribut d'instance** | Variable propre à une instance, créée via `self.nom`. |
| **attribut de classe** | Variable partagée par toutes les instances, déclarée dans le corps de la classe. |
| **méthode d'instance** | Méthode recevant `self`. Accède à l'état de l'instance. |
| **`@classmethod`** | Méthode recevant `cls` (la classe). Utilisée pour les factory methods. |
| **`@staticmethod`** | Méthode sans `self` ni `cls`. Utilitaire logiquement lié à la classe. |
| **héritage** | Mécanisme permettant à une classe d'hériter des attributs et méthodes d'une autre. |
| **sous-classe** | Classe qui hérite d'une autre. Peut surcharger ou étendre ses méthodes. |
| **`super()`** | Accède à la classe parente selon le MRO. Utilisé pour appeler le `__init__` ou une méthode parente. |
| **surcharge (override)** | Redéfinir dans une sous-classe une méthode héritée. |
| **MRO** | Method Resolution Order — ordre dans lequel Python cherche une méthode dans la hiérarchie de classes (algorithme C3). |
| **mixin** | Classe conçue pour ajouter des fonctionnalités via héritage multiple, sans être instanciée seule. |
| **duck typing** | Style Python : un objet est utilisable s'il possède les méthodes requises, quelle que soit sa classe. |
| **dunder** | Double underscore. Les méthodes `__xxx__` sont des méthodes spéciales interprétées par Python. |
| **`__repr__`** | Représentation développeur — utilisée dans le REPL, les logs, et `repr()`. |
| **`__str__`** | Représentation utilisateur — utilisée par `print()` et `str()`. |
| **`@property`** | Transforme une méthode en attribut en lecture (avec setter/deleter optionnels). |
| **name mangling** | `__attr` est renommé en `_Classe__attr` — évite les conflits de nom en héritage. |
| **`__slots__`** | Restreint les attributs autorisés sur une instance. Réduit la consommation mémoire. |
| **ABC** | Abstract Base Class (`abc.ABC`). Classe ne pouvant pas être instanciée directement — force l'implémentation dans les sous-classes. |
| **`@abstractmethod`** | Déclare une méthode abstraite — les sous-classes doivent l'implémenter. |
| **`Protocol`** | Définit un contrat structurel (duck typing formel) vérifié par les outils de type statique. |
| **`@dataclass`** | Décorateur générant automatiquement `__init__`, `__repr__`, `__eq__` à partir des annotations. |
| **composition** | Inclure un objet d'une autre classe comme attribut — alternative à l'héritage pour la réutilisation. |
