#python #pydantic #annotated #types #literal #union

## Annotated — types réutilisables

```python
from typing import Annotated
from pydantic import BaseModel, Field

Age         = Annotated[int,   Field(ge=0, le=120)]
Score       = Annotated[float, Field(ge=0.0, le=100.0)]
ShortStr    = Annotated[str,   Field(min_length=1, max_length=50)]
PositiveInt = Annotated[int,   Field(gt=0)]

class User(BaseModel):
    age:      Age
    score:    Score
    username: ShortStr
```

## Types réseau

```python
from pydantic.networks import EmailStr, AnyHttpUrl, IPvAnyAddress

class Contact(BaseModel):
    email:   EmailStr        # valide le format email
    website: AnyHttpUrl      # URL HTTP ou HTTPS
    ip:      IPvAnyAddress   # IPv4 ou IPv6

# EmailStr → pip install "pydantic[email]"
```

## Literal et Enum

```python
from typing import Literal
from enum import Enum

class Status(str, Enum):
    pending   = "pending"
    shipped   = "shipped"
    delivered = "delivered"

class Order(BaseModel):
    status: Status = Status.pending
    method: Literal["card", "paypal", "crypto"] = "card"

o = Order(status="shipped")   # ✅ coercition str → Enum
o.model_dump()                # {"status": "shipped", "method": "card"}
```

## Discriminated Union

```python
from typing import Union, Literal, Annotated

class Cat(BaseModel):
    type:  Literal["cat"]
    meows: int

class Dog(BaseModel):
    type:  Literal["dog"]
    barks: int

class Zoo(BaseModel):
    pet: Annotated[Union[Cat, Dog], Field(discriminator="type")]

Zoo(pet={"type": "cat", "meows": 5})   # ✅ → Cat
Zoo(pet={"type": "dog", "barks": 3})   # ✅ → Dog
```

> [!tip] Discriminated Union vs Union simple Avec un `discriminator`, Pydantic trouve le bon modèle en O(1). Indispensable pour les APIs polymorphiques (events, webhooks, payloads).
