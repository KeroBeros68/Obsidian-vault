#ia #constrained-decoding #langchain #dspy #intégration #pratique

## Intégration LangChain et DSPy

Le constrained decoding s'intègre naturellement dans les pipelines LangChain et DSPy pour garantir des sorties structurées dans des architectures complexes.

## LangChain + Outlines (modèle local)

```python
from langchain_community.llms import Outlines
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel
from typing import Literal, List

# Définir le schéma
class AnalyseTicket(BaseModel):
    catégorie: Literal["livraison", "retour", "paiement", "autre"]
    priorité:  Literal["urgente", "haute", "normale", "faible"]
    résumé:    str
    actions:   List[str]

# LLM avec Outlines via LangChain
llm = Outlines(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    json=AnalyseTicket   # ← contrainte JSON directement dans le LLM
)

# Chain LangChain standard
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un expert en support client. Analyse les tickets."),
    ("human", "Analyse ce ticket : {ticket}")
])

parser = PydanticOutputParser(pydantic_object=AnalyseTicket)

chain = prompt | llm | parser

résultat = chain.invoke({"ticket": "Mon colis n'est pas arrivé depuis 3 semaines !"})
print(résultat.catégorie)   # → "livraison"
print(résultat.priorité)    # → "haute"
```

## LangChain + vLLM guided decoding

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from pydantic import BaseModel
from typing import Literal, List
import json

class AnalyseTicket(BaseModel):
    catégorie: Literal["livraison", "retour", "paiement", "autre"]
    priorité:  Literal["urgente", "haute", "normale", "faible"]
    résumé:    str
    actions:   List[str]

# LLM = vLLM avec guided decoding
llm = ChatOpenAI(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    base_url="http://localhost:8000/v1",
    api_key="no-key",
    model_kwargs={
        "extra_body": {
            "guided_json": AnalyseTicket.model_json_schema(),
            "guided_decoding_backend": "xgrammar"
        }
    }
)

# with_structured_output() sur un LLM avec guided decoding = double garantie
llm_structuré = llm.with_structured_output(AnalyseTicket)

prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu analyses des tickets de support client."),
    ("human", "Analyse ce ticket : {ticket}")
])

chain = prompt | llm_structuré

résultat = chain.invoke({"ticket": "Colis cassé à la livraison."})
print(type(résultat))       # → <class 'AnalyseTicket'>
print(résultat.catégorie)   # → "retour"
```

## RAG + Constrained Decoding

Combiner RAG et constrained decoding pour des réponses à la fois sourcées ET structurées.

```python
from langchain_chroma import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
from pydantic import BaseModel
from typing import List, Optional

# Schéma RAG structuré
class RéponseRAG(BaseModel):
    réponse:    str
    sources:    List[str]
    confiance:  float
    info_manquante: Optional[str] = None

# LLM avec guided decoding (vLLM)
llm = ChatOpenAI(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    base_url="http://localhost:8000/v1",
    api_key="no-key",
    model_kwargs={"extra_body": {"guided_json": RéponseRAG.model_json_schema()}}
)

# Vectorstore
embeddings   = HuggingFaceEmbeddings(model_name="paraphrase-multilingual-MiniLM-L12-v2")
vectorstore  = Chroma(persist_directory="./db", embedding_function=embeddings)
retriever    = vectorstore.as_retriever(search_kwargs={"k": 4})

prompt = ChatPromptTemplate.from_messages([
    ("system", """Tu réponds en JSON structuré depuis le contexte fourni.
Contexte : {contexte}"""),
    ("human", "{question}")
])

def formater(docs):
    return "\n\n".join([f"[{d.metadata.get('source','?')}] {d.page_content}" for d in docs])

chain = (
    {"contexte": retriever | formater, "question": RunnablePassthrough()}
    | prompt
    | llm.with_structured_output(RéponseRAG)
)

résultat = chain.invoke("Quel est le délai de retour ?")
print(résultat.réponse)   # → "30 jours après réception"
print(résultat.sources)   # → ["politique.txt"]
print(résultat.confiance) # → 0.95
```

## DSPy + Constrained Decoding via vLLM

Utiliser DSPy avec un backend vLLM qui a le guided decoding activé.

```python
import dspy
from pydantic import BaseModel
from typing import Literal, List

# Configurer DSPy avec vLLM (qui a XGrammar activé)
lm = dspy.LM(
    "openai/mistralai/Mistral-7B-Instruct-v0.3",
    api_base="http://localhost:8000/v1",
    api_key="no-key"
)
dspy.configure(lm=lm)

# Dans DSPy, utiliser with_structured_output() via le LM
# Ou utiliser des signatures Pydantic + DSPy

class SignatureAnalyse(dspy.Signature):
    """Analyse structurée d'un ticket de support."""
    ticket:    str = dspy.InputField()
    catégorie: Literal["livraison", "retour", "paiement", "autre"] = dspy.OutputField()
    priorité:  Literal["urgente", "haute", "normale", "faible"] = dspy.OutputField()
    résumé:    str = dspy.OutputField(desc="Résumé en 1 phrase")
    actions:   List[str] = dspy.OutputField(desc="Liste d'actions à effectuer")

class ClassifieurDSPy(dspy.Module):
    def __init__(self):
        self.classifieur = dspy.ChainOfThought(SignatureAnalyse)

    def forward(self, ticket: str):
        return self.classifieur(ticket=ticket)

classifieur = ClassifieurDSPy()
résultat = classifieur(ticket="URGENT: Colis perdu depuis 3 semaines, client VIP !")
print(résultat.catégorie)   # → "livraison"
print(résultat.priorité)    # → "urgente"
```

## Comparaison des approches pour la sortie structurée

```
API Cloud (Claude, GPT-4) :
  llm.with_structured_output(MonSchéma)
  → Très fiable (~95%), pas de modèle local nécessaire
  → Coût par token, pas de contrôle sur le mécanisme

Modèle local + Outlines :
  outlines.generate.json(model, MonSchéma)
  → Garantie à 100%, gratuit à l'inférence
  → Nécessite modèle local + GPU

Modèle local + vLLM + XGrammar :
  guided_json + guided_decoding_backend=xgrammar
  → Garantie à 100%, le plus rapide en production
  → Nécessite vLLM server + GPU

Modèle local + Transformers + LM Format Enforcer :
  build_transformers_prefix_allowed_tokens_fn
  → Garantie à 100%, flexible pour contraintes dynamiques
  → Plus lent que XGrammar
```

> [!tip] Double garantie en production
> En production critique, combine `with_structured_output()` de LangChain (validation Pydantic) avec `guided_json` de vLLM (constrained decoding). Double couche de validation — quasi-zéro risque de sortie invalide.
