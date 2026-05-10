#python #pydantic #model_validator #validation #cross-field

## mode="after" — accès à l'objet complet

```python
from pydantic import BaseModel, model_validator

class Event(BaseModel):
    start: int
    end:   int

    @model_validator(mode="after")
    def check_order(self) -> "Event":
        if self.end <= self.start:
            raise ValueError("end doit être > start")
        return self   # ← OBLIGATOIRE

class PasswordForm(BaseModel):
    password: str
    confirm:  str

    @model_validator(mode="after")
    def passwords_match(self) -> "PasswordForm":
        if self.password != self.confirm:
            raise ValueError("Les mots de passe ne correspondent pas")
        return self
```

## mode="before" — transformer les données brutes

```python
@model_validator(mode="before")
@classmethod
def normalize_input(cls, data: dict) -> dict:
    if isinstance(data, dict):
        if "full_name" in data and "name" not in data:
            data["name"] = data.pop("full_name")
    return data
```

## Enrichir le modèle — calcul dans mode="after"

```python
class Invoice(BaseModel):
    price_ht: float
    tva:      float = 0.20
    total:    float = 0.0

    @model_validator(mode="after")
    def compute_total(self) -> "Invoice":
        self.total = round(self.price_ht * (1 + self.tva), 2)
        return self
```

## Tableau comparatif

||`@field_validator`|`@model_validator(mode="after")`|
|---|---|---|
|Accès|1 champ à la fois|tous les champs via `self`|
|Méthode|`@classmethod`|méthode d'instance|
|Retour|valeur transformée|`self`|
|Ordre|avant model_validator|après tous les field_validators|

## Ordre d'exécution complet

```
@model_validator(mode="before")
  → @field_validator(mode="before")  (ordre déclaration)
  → coercition de type
  → @field_validator(mode="after")   (ordre déclaration)
  → @model_validator(mode="after")
```

> [!warning] Retourner self est obligatoire en mode="after" Si tu oublies `return self`, le modèle vaut `None` → bug silencieux difficile à tracer.
