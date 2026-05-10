#python #pydantic #field_validator #validation #custom

## Syntaxe de base

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    username: str
    email:    str

    @field_validator("username")
    @classmethod
    def username_no_spaces(cls, v: str) -> str:
        if " " in v:
            raise ValueError("Le username ne peut pas contenir d'espaces")
        return v.lower()   # transformer + valider en une fois
```

## Plusieurs champs avec un seul décorateur

```python
@field_validator("email", "username")
@classmethod
def must_not_be_empty(cls, v: str) -> str:
    if not v.strip():
        raise ValueError("Ne peut pas être vide")
    return v.strip()
```

## Modes d'exécution

|Mode|Reçoit|@classmethod|Utile pour|
|---|---|---|---|
|`"before"`|valeur brute (non typée)|✅ requis|nettoyer avant cast|
|`"after"` (défaut)|valeur déjà typée|✅ requis|valider après conversion|
|`"wrap"`|valeur + handler|✅ requis|contrôle total avec fallback|

```python
@field_validator("name", mode="before")
@classmethod
def strip_name(cls, v):
    return str(v).strip()

@field_validator("name", mode="wrap")
@classmethod
def wrap_example(cls, v, handler):
    try:
        return handler(v)
    except Exception:
        return "default"   # fallback personnalisé
```

## Accéder aux champs précédents — info.data

```python
from pydantic import ValidationInfo

class Registration(BaseModel):
    username: str
    password: str
    confirm:  str

    @field_validator("confirm")
    @classmethod
    def passwords_match(cls, v: str, info: ValidationInfo) -> str:
        if "password" in info.data and v != info.data["password"]:
            raise ValueError("Les mots de passe ne correspondent pas")
        return v
    # ⚠️ info.data ne contient que les champs DÉJÀ validés (avant confirm)
```

> [!warning] @classmethod obligatoire En Pydantic v2, `@field_validator` **doit** toujours être accompagné de `@classmethod`.

> [!tip] Transformer ou valider ? Le validateur peut **retourner une valeur modifiée** (`.lower()`, `.strip()`, formatage). Ce n'est pas seulement un contrôle — c'est aussi un transformateur.
