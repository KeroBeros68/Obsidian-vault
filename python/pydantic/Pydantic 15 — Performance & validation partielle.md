#python #pydantic #performance #optimisation #partiel

## model_construct — bypasser la validation

```python
# Créer une instance SANS validation — ultra rapide
user = User.model_construct(name="Alice", age=30)
# ⚠️ Aucune validation — à réserver aux données internes de confiance
```

> [!warning] model_construct est dangereux Uniquement pour les données déjà validées (BDD, cache interne). Sur des données externes → toujours `model_validate()`.

## TypeAdapter — réutiliser le validateur en boucle

```python
from pydantic import TypeAdapter

ta = TypeAdapter(list[User])   # créer une fois, réutiliser partout

# En boucle intensive
users = ta.validate_python(raw_data_list)   # ✅ rapide
```

## Inspecter les erreurs — ValidationError

```python
from pydantic import ValidationError

try:
    User.model_validate(bad_data)
except ValidationError as e:
    print(e.error_count())                    # nb d'erreurs
    for err in e.errors(include_url=False):
        print(err["loc"], err["msg"], err["type"])
```

## model_fields — introspection

```python
for name, field in User.model_fields.items():
    print(name, field.annotation, field.is_required())

User.model_fields["name"].is_required()   # True
```

## Checklist performance

- [ ] `TypeAdapter` créé une fois, réutilisé en boucle
- [ ] `model_construct()` uniquement pour les données de confiance
- [ ] `model_dump(exclude_none=True)` réduit la taille des payloads
- [ ] `frozen=True` rend l'objet hashable → utilisable en set/cache
- [ ] `str_strip_whitespace=True` évite les field_validators redondants
- [ ] Settings avec `@lru_cache` — voir [[Pydantic 13 — pydantic-settings & config d'app]]
- [ ] Préférer `model_validate_json()` à `json.loads` + `model_validate`
- [ ] `exclude_unset=True` dans les PATCH pour des updates minimaux
