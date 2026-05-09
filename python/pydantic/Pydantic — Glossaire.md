#pydantic #glossaire #référence

|Terme|Définition|
|---|---|
|**BaseModel**|Classe de base — à hériter pour créer un modèle validé|
|**RootModel**|Modèle à valeur unique, sans champs nommés|
|**TypeAdapter**|Valide n'importe quel type Python sans créer de BaseModel|
|**ValidationError**|Exception levée quand les données ne respectent pas le schéma|
|**Field()**|Ajoute contraintes et métadonnées à un champ|
|**field_validator**|Validation personnalisée sur un champ|
|**model_validator**|Validation du modèle entier (règles multi-champs)|
|**field_serializer**|Sérialisation personnalisée d'un champ|
|**model_serializer**|Sérialisation personnalisée du modèle entier|
|**computed_field**|Champ calculé inclus dans model_dump()|
|**AfterValidator**|Validateur appliqué après la coercition de type|
|**BeforeValidator**|Validateur appliqué avant la coercition de type|
|**ConfigDict**|Objet de configuration du comportement d'un modèle (v2)|
|**model_config**|Attribut de classe qui reçoit le ConfigDict|
|**coercition**|Conversion automatique de type : `"30"` → `30`|
|**strict**|Mode sans coercition — types exacts requis|
|**frozen**|Rend l'objet immuable et hashable|
|**extra**|Comportement pour les champs inconnus : ignore / allow / forbid|
|**alias**|Nom alternatif d'un champ pour la (dé)sérialisation|
|**validation_alias**|Alias utilisé uniquement en entrée|
|**serialization_alias**|Alias utilisé uniquement en sortie|
|**alias_generator**|Fonction auto-appliquée à tous les champs|
|**AliasPath**|Alias qui extrait une valeur depuis un chemin imbriqué|
|**AliasChoices**|Alias qui accepte plusieurs noms alternatifs en entrée|
|**model_dump()**|Exporte en dict Python|
|**model_dump_json()**|Exporte en JSON string|
|**model_validate()**|Crée depuis un dict Python|
|**model_validate_json()**|Crée depuis une JSON string|
|**model_construct()**|Crée SANS validation (données de confiance uniquement)|
|**model_copy()**|Copie avec modifications|
|**model_fields_set**|Champs explicitement fournis à la création|
|**model_extra**|Champs non déclarés si extra="allow"|
|**from_attributes**|Crée depuis un objet ORM (attributs Python)|
|**BaseSettings**|Sous-classe pour lire la config depuis l'environnement|
|**populate_by_name**|Accepte nom Python ET alias en entrée|
|**validate_assignment**|Valide à chaque réassignation d'attribut|
|**Literal**|Type restreignant à des valeurs fixes|
|**Discriminated Union**|Union différenciée par un champ commun|
|**lru_cache**|Mise en cache des settings — charger une seule fois|
