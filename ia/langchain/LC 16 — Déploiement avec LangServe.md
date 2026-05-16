#ia #langchain #langserve #déploiement #production #avancé

## Déploiement avec LangServe

LangServe transforme n'importe quelle chain LangChain en API REST avec streaming, batch et playground intégrés.

## Installation

```bash
pip install "langserve[all]"
pip install fastapi uvicorn
```

## Serveur basique

```python
# serve.py
from fastapi import FastAPI
from langserve import add_routes
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

app = FastAPI(
    title="Mon API LangChain",
    version="1.0",
    description="API RAG et chatbot"
)

llm   = ChatAnthropic(model="claude-sonnet-4-20250514")
chain = (
    ChatPromptTemplate.from_messages([
        ("system", "Tu es un expert en {domaine}."),
        ("human", "{question}")
    ])
    | llm
    | StrOutputParser()
)

# Exposer la chain sous /chat
add_routes(
    app,
    chain,
    path="/chat",
    enabled_endpoints=["invoke", "stream", "batch", "playground"]
)

# Lancer : uvicorn serve:app --reload --port 8000
```

## Endpoints générés automatiquement

```
POST /chat/invoke        → appel synchrone
POST /chat/stream        → streaming SSE
POST /chat/batch         → batch parallèle
GET  /chat/playground    → UI interactive dans le navigateur
GET  /chat/input_schema  → schéma JSON d'entrée
GET  /chat/output_schema → schéma JSON de sortie
```

## Exposer plusieurs chains

```python
# Ajouter plusieurs routes
add_routes(app, chain_chat,  path="/chat")
add_routes(app, chain_rag,   path="/rag")
add_routes(app, agent,       path="/agent")

# → /chat/invoke, /rag/invoke, /agent/invoke
```

## Authentification

```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

bearer = HTTPBearer()

def vérifier_token(credentials: HTTPAuthorizationCredentials = Depends(bearer)):
    if credentials.credentials != "mon-token-secret":
        raise HTTPException(status_code=401, detail="Token invalide")

add_routes(
    app,
    chain,
    path="/chat",
    dependencies=[Depends(vérifier_token)]   # ← auth sur tous les endpoints
)
```

## Client Python pour consommer l'API

```python
from langserve import RemoteRunnable

# Se connecter à l'API déployée
chain_distante = RemoteRunnable("http://localhost:8000/chat")

# Utilisation identique à une chain locale !
réponse = chain_distante.invoke({
    "domaine": "Python",
    "question": "C'est quoi un générateur ?"
})

# Streaming depuis l'API
for chunk in chain_distante.stream({"domaine": "Python", "question": "..."}):
    print(chunk, end="", flush=True)

# Batch depuis l'API
réponses = chain_distante.batch([
    {"domaine": "Python", "question": "Q1"},
    {"domaine": "Python", "question": "Q2"},
])
```

## Déploiement Docker

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY serve.py .

EXPOSE 8000
CMD ["uvicorn", "serve:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t mon-api-langchain .
docker run -p 8000:8000 -e ANTHROPIC_API_KEY=ta-clé mon-api-langchain
```

> [!tip] Le playground est précieux
> L'endpoint `/playground` génère automatiquement une UI pour tester la chain dans le navigateur. Parfait pour les démonstrations et le debugging sans écrire de frontend.

> [!info] LangServe vs FastAPI custom
> LangServe accélère le déploiement initial. Pour des besoins très spécifiques (auth complexe, logique métier avancée), construire une API FastAPI custom sur la chain reste une bonne option.
