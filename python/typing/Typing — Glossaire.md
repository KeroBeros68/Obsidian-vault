#python #typing #glossaire #référence

|Terme|Définition|
|---|---|
|**annotation**|Indication de type associée à une variable, un paramètre ou un retour de fonction|
|**type hint**|Synonyme d'annotation de type|
|**Any**|Type spécial qui désactive toute vérification — compatible avec tout|
|**Optional[X]**|Équivalent de `X \| None` — la valeur peut être X ou None|
|**Union[X, Y]**|La valeur peut être X ou Y — `X \| Y` en Python 3.10+|
|**Literal**|Restreint à des valeurs exactes : `Literal["a", "b"]`|
|**Final**|Variable non réassignable — constante|
|**ClassVar**|Variable partagée entre toutes les instances d'une classe|
|**TypeVar**|Paramètre de type générique — `T = TypeVar("T")`|
|**Generic**|Classe paramétrée par un ou plusieurs TypeVar|
|**Protocol**|Interface structurelle — duck typing avec vérification statique|
|**runtime_checkable**|Décorateur qui permet `isinstance(obj, Protocol)`|
|**TypedDict**|Dict avec des clés et types fixes — vérifié statiquement|
|**NamedTuple**|Tuple avec des champs nommés et typés — immuable|
|**Callable**|Type d'une fonction : `Callable[[arg1, arg2], retour]`|
|**Iterator[T]**|Objet avec `__iter__` et `__next__` — produit des T|
|**Iterable[T]**|Objet avec `__iter__` — peut être parcouru|
|**Generator[Y, S, R]**|Générateur avec types de yield, send et return|
|**Annotated**|Attache des métadonnées à un type — `Annotated[int, Field(gt=0)]`|
|**NotRequired**|Clé optionnelle dans un TypedDict|
|**Self**|Représente le type de l'instance courante (Python 3.11+)|
|**NoReturn**|Fonction qui ne retourne jamais (lève toujours une exception)|
|**ParamSpec**|Capture la signature complète d'une fonction (pour les décorateurs)|
|**TypeVarTuple**|TypeVar variadique — pour les tuples de longueur variable|
|**@overload**|Déclare des signatures multiples pour une même fonction|
|**narrowing**|Rétrécissement du type dans une branche conditionnelle|
|**forward reference**|Type cité en string car pas encore défini : `"MyClass"`|
|**TYPE_CHECKING**|Booléen `False` au runtime, `True` dans le type checker — pour les imports conditionnels|
|**mypy**|Vérificateur de types statique de référence pour Python|
|**pyright**|Vérificateur de types de Microsoft — rapide, intégré à Pylance/VSCode|
|**reveal_type**|Fonction magique du type checker pour inspecter le type inféré|
|**type: ignore**|Commentaire pour supprimer une erreur mypy sur une ligne|
|**cast**|Indique un type au type checker sans vérification runtime|
|**stub (.pyi)**|Fichier d'annotations de types pour une librairie sans annotations|
|**coercition**|Conversion de type — Python le fait, les annotations ne le font pas|
|**duck typing**|"Si ça marche comme un canard..." — typage par comportement, pas par héritage|
