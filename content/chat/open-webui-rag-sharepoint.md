# Rag System for Open-WebUI and Sharepoint

> Build a system with Open-WebUI and a RAG with documents in SharePoint 

Excellent use case — and very doable.

But I want to be very direct:

> **Do not use vanilla Open-WebUI Knowledge upload as your primary SharePoint enterprise RAG ingestion layer.**

Open WebUI’s built-in Knowledge/RAG is good for manual document collections, but it becomes limiting when you need: scheduled syncs, permissions, metadata filters, delta updates, tenant auth, large corpora, or production-grade retrieval tuning. Even the community repeatedly hits these limitations when trying to automate enterprise knowledge ingestion. ([Open WebUI][1])

What you actually want is:

> **Open-WebUI as the chat frontend**
>
> * **Custom RAG backend service**
> * **SharePoint ingestion/indexing pipeline**
> * **Vector DB + metadata-aware retrieval**

That architecture scales.

---

# 1. First decide the architecture (this is the critical fork)

You have two possible ways:

---

## OPTION A — Quick & Dirty (not ideal long term)

Use Open-WebUI Knowledge Base API and keep uploading processed SharePoint files into it.

Flow:

SharePoint → Download docs → Convert text → Chunk → Upload into OpenWebUI Knowledge

Pros:

* easy
* mostly no frontend coding
* uses native OpenWebUI KB UI

Cons:

* weak metadata filtering
* weak ACL/security mapping
* limited ingestion control
* painful for 10k+ docs
* difficult delta sync
* difficult source traceability

---

## OPTION B — Professional Enterprise RAG (recommended)

Use Open-WebUI only as the LLM chat UI, but retrieval comes from your own API.

Flow:

```text
User asks in Open-WebUI
        ↓
Open-WebUI calls custom Pipeline / Function / Tool
        ↓
Your RAG API searches vector DB
        ↓
RAG API has indexed SharePoint docs + metadata + permissions
        ↓
Top chunks returned
        ↓
LLM answers with citations
```

This is how most serious deployments do it.

---

# 2. High-level system you should build

Here is the production-grade blueprint:

```text
[ SharePoint Online ]
      ↓ Microsoft Graph API
[ Ingestion Worker ]
      ↓
[ Document Parser ]
      ↓
[ Chunking + Metadata Builder ]
      ↓
[ Embedding Service ]
      ↓
[ Vector Database ]
      ↓
[ Retrieval API / RAG Service ]
      ↓
[ Open-WebUI Pipeline / Tool ]
      ↓
[ User Chat ]
```

---

# 3. Component-by-component implementation

---

## COMPONENT A — Connect to SharePoint properly

Do NOT scrape SharePoint pages.

Use:

Microsoft Graph API

Specifically:

* `/sites`
* `/drives`
* `/drive/items`
* `/lists`
* `/content`

You create an Azure App Registration with:

* Sites.Read.All
* Files.Read.All
* offline_access

Then use OAuth client credentials.

This gives you:

* all document libraries
* file metadata
* modified timestamps
* authors
* folders
* permissions
* file ids for delta sync

This is enterprise-safe.

---

## COMPONENT B — Build an ingestion daemon (scheduled sync)

You need a Python microservice that runs every X minutes.

It should:

### Pull only changed files

Using Graph delta endpoints:

```http
/drives/{drive-id}/root/delta
```

Meaning:

* new docs
* updated docs
* deleted docs

No need to reindex everything.

This is essential once you hit hundreds/thousands of files.

---

## COMPONENT C — Parse documents intelligently

SharePoint docs may be:

* PDF
* DOCX
* PPTX
* XLSX
* TXT
* HTML
* OneNote export
* emails

Use:

* pymupdf
* python-docx
* openpyxl
* unstructured.io
* tika (optional)

Extract:

```python
{
  "doc_id": "...",
  "title": "...",
  "site": "...",
  "library": "...",
  "folder": "...",
  "author": "...",
  "modified": "...",
  "sharepoint_url": "...",
  "access_groups": [...],
  "raw_text": "..."
}
```

Metadata matters enormously.

Most failed RAG systems are actually metadata-poor systems.

---

## COMPONENT D — Chunking (this is where many people mess up)

Do not blindly chunk by 500 chars.

Use semantic/document-aware chunking:

### Recommended:

* chunk by headings
* preserve section titles
* 600–1200 token chunks
* 10–15% overlap

Each chunk should carry metadata:

```python
{
  "chunk_id": "...",
  "doc_id": "...",
  "section": "Vacation Policy > Sick Leave",
  "site": "HR Portal",
  "modified": "...",
  "permissions": [...],
  "text": "..."
}
```

Open-WebUI itself exposes chunking settings, but for enterprise SharePoint corpora custom preprocessing gives far more control and consistency. ([Open WebUI][2])

---

## COMPONENT E — Embeddings

Use a strong embedding model.

Recommended today:

* OpenAI `text-embedding-3-large`
  or
* Cohere embed-v4
  or
* BGE-large if self-hosted

Avoid weak MiniLM embeddings for enterprise docs.

This single decision changes retrieval quality massively.

---

## COMPONENT F — Vector DB

Use one of:

* Qdrant  ← my top recommendation
* pgvector
* Weaviate

Store:

* embedding
* text
* metadata
* ACL fields
* source URL

Need metadata filtering like:

```python
site = HR
AND access_group contains FinanceTeam
AND modified > 2025-01-01
```

This is huge.

---

## COMPONENT G — Retrieval service (your custom RAG brain)

Build a FastAPI service:

Endpoints:

```text
POST /search
POST /ask
POST /reindex
POST /delta-sync
```

Retrieval pipeline:

### Query flow:

1. embed user question
2. vector search top 20
3. BM25 keyword search top 20
4. hybrid rerank
5. ACL filter
6. return top 5 chunks

This hybrid retrieval consistently beats pure vector search.

Open-WebUI now supports hybrid retrieval internally too, which confirms this direction, but custom implementation gives you better enterprise control. ([Open WebUI][1])

---

## COMPONENT H — Connect this to Open-WebUI

This is the nice part:

Open-WebUI supports:

* Pipelines
* Tools
* Functions
* Custom API integrations

So instead of native Knowledge:

you create:

> `search_sharepoint_knowledge(query, user_identity)`

Open-WebUI calls your backend.

Backend returns:

```json
{
  "sources": [
    {
      "title": "HR Vacation Policy",
      "url": "...",
      "snippet": "Employees are entitled to..."
    }
  ],
  "context": "..."
}
```

Then LLM answers grounded on that.

---

# 4. VERY important: permissions / security trimming

If this is for a company:

you cannot allow everyone to retrieve all SharePoint docs.

You must carry SharePoint ACLs during ingestion.

Meaning each chunk stores:

```python
allowed_groups = ["HR", "Finance", "Managers"]
```

Then when user chats:

Open-WebUI passes username/email/group claim.

Retrieval API filters unauthorized chunks.

Without this, legal/security will shut down the project.

---

# 5. Suggested tech stack (battle-tested)

Here’s what I would personally build in 2026:

| Layer             | Recommendation                 |
| ----------------- | ------------------------------ |
| Frontend Chat     | Open-WebUI                     |
| Auth              | Azure Entra ID SSO             |
| SharePoint Access | Microsoft Graph API            |
| Ingestion Worker  | Python + Celery/APScheduler    |
| Parsing           | unstructured + pymupdf + docx  |
| Embeddings        | text-embedding-3-large         |
| Vector DB         | Qdrant                         |
| RAG API           | FastAPI                        |
| Reranking         | Cohere rerank or bge-reranker  |
| LLM               | GPT-4.1 / Claude / local Llama |

This is a serious enterprise architecture.

---

# 6. Realistic build phases (do this in order)

---

## Phase 1 — Proof of Concept (1 week)

* ingest one SharePoint library
* parse PDFs/DOCX
* Qdrant vector db
* simple retrieval API
* Open-WebUI function call

---

## Phase 2 — Proper RAG (2–3 weeks)

* metadata chunking
* hybrid search
* citations
* source links
* better prompting
* answer grounding

---

## Phase 3 — Enterprise Ready

* delta sync
* ACL filtering
* multi-library support
* usage logging
* feedback loop
* hallucination scoring

---

# 7. My honest recommendation

Because this project has quite a few moving parts, the biggest mistake is trying to improvise it ad hoc.

You need:

* ingestion code
* Graph auth
* parser layer
* vector schema
* retrieval API
* Open-WebUI integration code

all designed coherently.

---

# I can do something very useful next:

I can provide you a **full implementation blueprint with folder structure + Python services + sample code for every component**
(almost like an engineer's build manual).

Just say:

> continue

[1]: https://docs.openwebui.com/features/workspace/knowledge/?utm_source=chatgpt.com "Knowledge / Open WebUI"
[2]: https://docs.openwebui.com/features/chat-conversations/rag/?utm_source=chatgpt.com "Retrieval Augmented Generation (RAG) / Open WebUI"


# Use it with docling


Perfect — that actually upgrades the architecture significantly.

**Docling is currently one of the best open-source parsers specifically built for GenAI/RAG ingestion**, with structured extraction for PDF, DOCX, PPTX, XLSX, HTML, OCR docs, reading order, tables, formulas, and markdown/JSON export. It was designed exactly for this class of pipeline. ([GitHub][1])

And yes:

> if we use Docling, the SharePoint → RAG pipeline becomes much cleaner than with traditional parser stacks.

---

# Revised enterprise architecture (Docling-first)

This becomes:

```text
SharePoint Online
    ↓ Graph API Delta Sync
Ingestion Worker
    ↓
Docling Parsing Engine
    ↓
Docling Structured Document Model
    ↓
Semantic Chunk Builder
    ↓
Embedding Generator
    ↓
Qdrant / pgvector
    ↓
Retrieval API
    ↓
Open-WebUI Pipeline Tool
    ↓
LLM Response with citations
```

---

# Why Docling is the right choice here

Traditional parsers:

* `python-docx`
* `pymupdf`
* `pdfplumber`
* `openpyxl`

all give you fragmented low-level text.

Docling gives:

* document structure
* headings
* tables
* lists
* page boundaries
* OCR
* reading order
* markdown
* JSON document tree
* image blocks
* semantic components

This is exactly what RAG needs.

Community consensus has also shifted heavily toward Docling for self-hosted enterprise parsing, especially because it unifies many office formats under one API. ([Reddit][2])

---

# IMPORTANT DESIGN CHANGE:

## We should not store flat text from SharePoint.

## We should store Docling-structured chunks.

This is a huge difference.

Instead of:

```python
raw_text = extract_text(file)
chunks = naive_split(raw_text)
```

we do:

```python
docling_doc = converter.convert(file).document
structured_nodes = docling_doc.export_to_dict()
chunks = semantic_chunk_docling(structured_nodes)
```

Now your chunks preserve:

* section titles
* bullet hierarchy
* table association
* nearby captions
* page refs
* source coordinates

This dramatically improves retrieval grounding.

---

# Full implementation blueprint (Docling version)

---

# MODULE 1 — SharePoint Connector Service

Python microservice:

`sharepoint_sync.py`

Responsibilities:

* authenticate to Microsoft Graph
* walk document libraries
* pull file metadata
* detect changes using delta endpoint
* download changed files to temp processing folder

Metadata captured:

```python
{
    "sp_file_id": "...",
    "site_name": "...",
    "library_name": "...",
    "folder_path": "...",
    "filename": "...",
    "last_modified": "...",
    "sharepoint_url": "...",
    "permissions": [...]
}
```

---

# MODULE 2 — Docling Parse Service

Install:

```bash
pip install docling
```

Docling supports direct AI-oriented conversion to structured markdown/json and many enterprise file types. ([Docling][3])

Basic parser:

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()

result = converter.convert("/tmp/employee_handbook.pdf")
doc = result.document

markdown = doc.export_to_markdown()
json_doc = doc.export_to_dict()
```

---

## Why we use BOTH markdown + json

### markdown

good for human-readable fallback and debugging

### json_doc

goldmine for semantic chunking

because JSON contains:

* titles
* text blocks
* tables
* page refs
* hierarchy

---

# MODULE 3 — Custom Docling Semantic Chunker

This is where most of the RAG intelligence lives.

Do NOT use generic character splitters.

We build:

`docling_chunker.py`

Logic:

### chunk by document hierarchy:

* Heading
* Subheading
* Paragraph group
* Table + caption
* Bullet list section

Pseudo:

```python
def build_chunks(docling_json, metadata):
    chunks = []

    current_section = []
    current_title = "ROOT"

    for node in docling_json["body"]["children"]:

        if node["label"] in ["section_header", "title", "heading"]:
            if current_section:
                chunks.append(make_chunk(current_title, current_section, metadata))
            current_title = node["text"]
            current_section = []

        else:
            current_section.append(node)

    if current_section:
        chunks.append(make_chunk(current_title, current_section, metadata))

    return chunks
```

Every chunk stores:

```python
{
    "chunk_id": uuid,
    "doc_id": sp_file_id,
    "title": current_title,
    "site": site_name,
    "library": library_name,
    "folder": folder_path,
    "sharepoint_url": sharepoint_url,
    "permissions": permissions,
    "content": rendered_text,
    "modified": last_modified
}
```

---

# MODULE 4 — Table Preservation Strategy (very important)

Docling preserves tables structurally.

Normal parsers destroy tables into nonsense text.

For enterprise SharePoint docs:

* policy docs
* financial reports
* project plans
* KPI sheets

this matters a lot.

When node type = table:

convert table into markdown table inside chunk.

Example:

```python
| Department | Vacation Days |
|------------|---------------|
| HR         | 25            |
| IT         | 28            |
```

This makes LLM retrieval far more accurate.

---

# MODULE 5 — Embedding Worker

Then embed each chunk.

Recommended:

* OpenAI text-embedding-3-large
  or
* local BGE-M3

Store vectors + metadata in:

Qdrant

Best fit because:

* metadata filters
* fast cosine search
* payload storage
* ACL filtering

---

# MODULE 6 — Retrieval API

FastAPI app:

```python
POST /retrieve
POST /ask
POST /sync-sharepoint
```

Query flow:

```text
user question
   ↓
embedding query
   ↓
Qdrant vector search top 30
   ↓
keyword rerank
   ↓
metadata filter
   ↓
ACL filter
   ↓
top 5 chunks
```

---

# MODULE 7 — Open-WebUI Integration

This is the part many people misunderstand:

We do NOT upload docs manually into Open-WebUI knowledge.

Instead:

create an Open-WebUI custom Tool or Pipeline called:

```python
search_sharepoint_rag(query, user_email)
```

When user asks:

> "what is our PTO carryover policy?"

Open-WebUI sends query to your FastAPI RAG backend.

Backend returns:

```json
{
  "context": "...retrieved chunk text...",
  "sources": [
    {
      "title": "Employee Handbook",
      "url": "sharepoint-url",
      "section": "Paid Time Off > Carryover"
    }
  ]
}
```

LLM answers from that context.

---

# MODULE 8 — Docling Performance Reality (important warning)

Need to tell you this now:

Docling is excellent...

but CPU-only batch parsing of large SharePoint repositories can become slow.

People routinely report:

* 100 page docs taking minutes
* image-heavy docs taking much longer

especially with OCR enabled. ([Reddit][4])

So production setup should use:

### Fast lane / Slow lane parsing

---

## Fast lane (most docs)

Native digital docs:

* DOCX
* PPTX
* XLSX
* text PDFs

standard Docling parse

---

## Slow lane

Scanned PDFs
image-heavy PDFs
camera captures

Docling + OCR profile on worker queue

Asynchronous.

Do not block ingestion pipeline.

---

# MODULE 9 — Recommended deployment topology

```text
docker-compose
│
├── open-webui
├── qdrant
├── rag-api-fastapi
├── sharepoint-sync-worker
├── docling-parser-worker
├── redis
└── postgres(optional logs)
```

This is very maintainable.

---

# Honest answer:

Now that you've chosen Docling:

> this is no longer a simple "RAG demo"

this becomes a **very serious enterprise knowledge platform**, and that's good.

Because Docling gives us enough document intelligence to build something genuinely useful instead of a toy chatbot.

---

# Strong recommendation for next step

The next thing we should design is the **actual codebase**:

* folder structure
* Docker compose
* SharePoint Graph auth
* Docling parser service
* Qdrant schema
* FastAPI retrieval endpoints
* Open-WebUI pipeline plugin

I can generate the full production code skeleton.

Just say:

> continue with codebase

[1]: https://github.com/docling-project/docling?utm_source=chatgpt.com "GitHub - docling-project/docling: Get your documents ready for gen AI · GitHub"
[2]: https://www.reddit.com/r/Rag/comments/1oqrrpe/what_do_you_use_for_document_parsing_for/?utm_source=chatgpt.com "What do you use for document parsing for enterprise data ingestion?"
[3]: https://www.docling.ai/?utm_source=chatgpt.com "Docling"
[4]: https://www.reddit.com/r/LangChain/comments/1pn4pw5/whatre_you_using_for_pdf_parsing/?utm_source=chatgpt.com "What're you using for PDF parsing?"

# Code base
> continue with code base

Excellent.

We're now at the point where we move from architecture into **buildable engineering**.

I’m going to give you a **production-grade monorepo structure** that you can actually implement, not toy snippets.

This stack will be:

> Open-WebUI + SharePoint + Docling + Qdrant + FastAPI + Async Workers

and designed so you can deploy it in Docker.

---

# 1. Full repository layout

```text id="ftcbj5"
enterprise-sharepoint-rag/
│
├── docker-compose.yml
│
├── .env
│
├── services/
│   │
│   ├── rag_api/
│   │   ├── main.py
│   │   ├── routes/
│   │   │   ├── retrieve.py
│   │   │   ├── ask.py
│   │   │   └── sync.py
│   │   ├── core/
│   │   │   ├── embeddings.py
│   │   │   ├── retriever.py
│   │   │   ├── reranker.py
│   │   │   ├── acl.py
│   │   │   └── prompts.py
│   │   └── models/
│   │       └── schemas.py
│   │
│   ├── sharepoint_worker/
│   │   ├── sync_daemon.py
│   │   ├── graph_client.py
│   │   ├── delta_tracker.py
│   │   └── downloader.py
│   │
│   ├── parser_worker/
│   │   ├── parse_worker.py
│   │   ├── docling_parser.py
│   │   ├── docling_chunker.py
│   │   └── metadata_builder.py
│   │
│   ├── indexer_worker/
│   │   ├── embed_worker.py
│   │   ├── qdrant_client.py
│   │   └── upsert.py
│   │
│   └── openwebui_pipeline/
│       └── sharepoint_rag_pipeline.py
│
├── storage/
│   ├── incoming_docs/
│   ├── parsed_docs/
│   └── failed_docs/
│
└── requirements/
    ├── rag_api.txt
    ├── workers.txt
    └── pipeline.txt
```

This separation is important:

* sync worker = only talks to SharePoint
* parser worker = only parses docs
* indexer = only embeddings/vector db
* rag api = only retrieval
* OpenWebUI pipeline = only UI bridge

Loose coupling = maintainable.

---

# 2. Docker compose (deployment backbone)

```yaml id="z4ngm8"
version: '3.9'

services:

  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - ./qdrant_storage:/qdrant/storage

  rag_api:
    build: ./services/rag_api
    ports:
      - "8000:8000"
    depends_on:
      - qdrant

  sharepoint_worker:
    build: ./services/sharepoint_worker
    depends_on:
      - rag_api

  parser_worker:
    build: ./services/parser_worker

  indexer_worker:
    build: ./services/indexer_worker
    depends_on:
      - qdrant

  redis:
    image: redis:7

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    ports:
      - "3000:8080"
    volumes:
      - open-webui:/app/backend/data

volumes:
  open-webui:
```

Eventually we'll connect workers via Redis queue.

---

# 3. Environment file (.env)

```bash id="h5yzg8"
GRAPH_TENANT_ID=
GRAPH_CLIENT_ID=
GRAPH_CLIENT_SECRET=

SHAREPOINT_SITE_ID=
SHAREPOINT_DRIVE_ID=

OPENAI_API_KEY=

QDRANT_URL=http://qdrant:6333
RAG_API_URL=http://rag_api:8000
```

---

# 4. SharePoint Graph client

`services/sharepoint_worker/graph_client.py`

```python id="7ckxyc"
import requests
import msal
import os

TENANT_ID = os.getenv("GRAPH_TENANT_ID")
CLIENT_ID = os.getenv("GRAPH_CLIENT_ID")
CLIENT_SECRET = os.getenv("GRAPH_CLIENT_SECRET")

AUTHORITY = f"https://login.microsoftonline.com/{TENANT_ID}"
SCOPE = ["https://graph.microsoft.com/.default"]

class GraphClient:
    def __init__(self):
        app = msal.ConfidentialClientApplication(
            CLIENT_ID,
            authority=AUTHORITY,
            client_credential=CLIENT_SECRET,
        )
        token = app.acquire_token_for_client(scopes=SCOPE)
        self.headers = {"Authorization": f"Bearer {token['access_token']}"}

    def delta_files(self, drive_id, delta_link=None):
        if delta_link:
            url = delta_link
        else:
            url = f"https://graph.microsoft.com/v1.0/drives/{drive_id}/root/delta"
        return requests.get(url, headers=self.headers).json()

    def download_file(self, item_id, path):
        url = f"https://graph.microsoft.com/v1.0/drives/{os.getenv('SHAREPOINT_DRIVE_ID')}/items/{item_id}/content"
        r = requests.get(url, headers=self.headers)
        with open(path, "wb") as f:
            f.write(r.content)
```

---

# 5. Sync daemon

`sync_daemon.py`

```python id="yhqoq9"
import time
import uuid
import os
from graph_client import GraphClient

gc = GraphClient()

DELTA_LINK_FILE = "/data/delta_link.txt"

while True:
    delta_link = None
    if os.path.exists(DELTA_LINK_FILE):
        delta_link = open(DELTA_LINK_FILE).read().strip()

    result = gc.delta_files(os.getenv("SHAREPOINT_DRIVE_ID"), delta_link)

    for item in result.get("value", []):
        if "file" not in item:
            continue

        local_name = f"/storage/incoming_docs/{uuid.uuid4()}_{item['name']}"
        gc.download_file(item["id"], local_name)

        # send metadata manifest to parser queue
        print("Downloaded:", local_name)

    if "@odata.deltaLink" in result:
        open(DELTA_LINK_FILE, "w").write(result["@odata.deltaLink"])

    time.sleep(300)
```

This runs every 5 mins.

---

# 6. Docling parser worker

`docling_parser.py`

```python id="v2q8je"
from docling.document_converter import DocumentConverter

converter = DocumentConverter()

def parse_document(filepath):
    result = converter.convert(filepath)
    return result.document.export_to_dict(), result.document.export_to_markdown()
```

---

# 7. Semantic chunker (important core)

`docling_chunker.py`

```python id="e0y1r6"
import uuid

def walk_nodes(nodes, current_heading="ROOT"):
    chunks = []
    buffer = []

    for node in nodes:
        label = node.get("label", "")
        text = node.get("text", "")

        if label in ["heading", "title", "section_header"]:
            if buffer:
                chunks.append((current_heading, "\n".join(buffer)))
                buffer = []
            current_heading = text
        else:
            if text:
                buffer.append(text)

    if buffer:
        chunks.append((current_heading, "\n".join(buffer)))

    return chunks


def build_chunk_payloads(docling_json, metadata):
    body = docling_json.get("body", {}).get("children", [])
    raw_chunks = walk_nodes(body)

    payloads = []

    for heading, content in raw_chunks:
        payloads.append({
            "chunk_id": str(uuid.uuid4()),
            "doc_id": metadata["sp_file_id"],
            "title": heading,
            "content": content,
            "sharepoint_url": metadata["sharepoint_url"],
            "site": metadata["site_name"],
            "library": metadata["library_name"],
            "permissions": metadata["permissions"],
            "modified": metadata["last_modified"]
        })

    return payloads
```

---

# 8. Embedding + Qdrant upsert

`embed_worker.py`

```python id="0gc8z7"
from openai import OpenAI
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
qdrant = QdrantClient(url=os.getenv("QDRANT_URL"))

def embed(text):
    return client.embeddings.create(
        model="text-embedding-3-large",
        input=text
    ).data[0].embedding

def upsert_chunks(chunks):
    points = []
    for c in chunks:
        vec = embed(c["title"] + "\n" + c["content"])
        points.append(PointStruct(
            id=c["chunk_id"],
            vector=vec,
            payload=c
        ))
    qdrant.upsert(collection_name="sharepoint_knowledge", points=points)
```

---

# 9. Retrieval API

`services/rag_api/routes/retrieve.py`

```python id="0n8shg"
from fastapi import APIRouter
from pydantic import BaseModel
from qdrant_client import QdrantClient
from core.embeddings import embed_query
import os

router = APIRouter()
qdrant = QdrantClient(url=os.getenv("QDRANT_URL"))

class RetrieveRequest(BaseModel):
    query: str
    user_groups: list[str]

@router.post("/retrieve")
def retrieve(req: RetrieveRequest):
    vector = embed_query(req.query)

    hits = qdrant.search(
        collection_name="sharepoint_knowledge",
        query_vector=vector,
        limit=15
    )

    filtered = []
    for h in hits:
        perms = h.payload.get("permissions", [])
        if not perms or set(req.user_groups).intersection(set(perms)):
            filtered.append(h.payload)

    return filtered[:5]
```

---

# 10. Open-WebUI pipeline connector

Open-WebUI supports custom pipelines/functions for external retrieval grounding.

`sharepoint_rag_pipeline.py`

```python id="9mjlwm"
import requests
import os

class Pipeline:

    async def inlet(self, body: dict, user: dict = None):
        query = body["messages"][-1]["content"]

        groups = user.get("groups", []) if user else []

        r = requests.post(
            f"{os.getenv('RAG_API_URL')}/retrieve",
            json={
                "query": query,
                "user_groups": groups
            }
        )

        docs = r.json()

        context = "\n\n".join([
            f"[{d['title']}]\n{d['content']}" for d in docs
        ])

        body["messages"][-1]["content"] = f"""
Use ONLY the following SharePoint knowledge context to answer.

{context}

User question: {query}
"""
        return body
```

This is where Open-WebUI becomes enterprise-grounded.

---

# 11. Immediate engineering truth

This is now a legitimate production scaffold.

But there are still **four enterprise-critical things missing** before this is truly good:

### missing A — Redis task queue wiring

workers should not run as isolated scripts

### missing B — deletion handling

when SharePoint file deleted → remove vectors

### missing C — reranker

vector similarity alone is not enough

### missing D — citations/source formatting in Open-WebUI answers

Those four are what make the difference between "works" and "users trust it".

---

# 12. My recommendation: continue deeper

The next layer is where this becomes a serious deployable system:

> I can now generate the **actual enterprise-grade version** with:
>
> * async Redis/Celery queues
> * complete Dockerfiles
> * Qdrant collection schema
> * file deletion sync
> * reranking
> * source citations
> * Open-WebUI admin installation steps

This is the version you'd hand to engineers.

Just say:

> continue enterprise version

# Enterprise Grade
> continue with enterprise grade

Excellent — now we move from **prototype scaffold** to **deployable enterprise system**.

This is where most online tutorials stop, but this is actually where the hard engineering starts.

We are going to turn the previous monorepo into:

> **fault tolerant + asynchronous + recoverable + scalable + admin maintainable**

---

# ENTERPRISE DESIGN PRINCIPLES (important before code)

Your system now needs these guarantees:

### 1. SharePoint sync must never block parsing

### 2. Parsing must never block embedding

### 3. Failed docs must retry safely

### 4. Deleted docs must be removed from Qdrant

### 5. Open-WebUI queries must stay low latency

### 6. Source citations must be auditable

### 7. Workers must scale independently

That means:

> we must introduce a proper message queue.

---

# FINAL PRODUCTION ARCHITECTURE

```text id="smf8wh"
SharePoint Graph Poller
      ↓
Redis Task Queue
      ↓
Docling Parser Workers
      ↓
Redis Task Queue
      ↓
Embedding/Indexer Workers
      ↓
Qdrant Vector Store
      ↓
RAG FastAPI
      ↓
Open-WebUI Pipeline
```

---

# 1. Updated monorepo

```text id="3zcj7j"
enterprise-sharepoint-rag/
│
├── docker-compose.yml
├── .env
│
├── services/
│   ├── common/
│   │   ├── redis_queue.py
│   │   ├── models.py
│   │   └── logger.py
│   │
│   ├── sharepoint_worker/
│   │   ├── sync_daemon.py
│   │   ├── graph_client.py
│   │   └── deletion_handler.py
│   │
│   ├── parser_worker/
│   │   ├── parse_consumer.py
│   │   ├── docling_parser.py
│   │   ├── docling_chunker.py
│   │   └── manifest_writer.py
│   │
│   ├── indexer_worker/
│   │   ├── embed_consumer.py
│   │   ├── qdrant_indexer.py
│   │   ├── deletion_consumer.py
│   │   └── rerank_features.py
│   │
│   ├── rag_api/
│   │   ├── main.py
│   │   ├── routes/
│   │   ├── core/
│   │   └── citation_builder.py
│   │
│   └── openwebui_pipeline/
│       └── sharepoint_rag_pipeline.py
│
└── storage/
```

---

# 2. Redis queue backbone

Why Redis:

* simple
* durable enough
* Celery/RQ compatible
* Docker easy

Install:

```bash id="ov7qij"
pip install redis rq
```

---

## common/redis_queue.py

```python id="ww4n8v"
from redis import Redis
from rq import Queue
import os

redis_conn = Redis(host="redis", port=6379)
parse_queue = Queue("parse_queue", connection=redis_conn)
embed_queue = Queue("embed_queue", connection=redis_conn)
delete_queue = Queue("delete_queue", connection=redis_conn)
```

---

# 3. SharePoint sync worker now becomes producer only

Very important design shift:

Sync worker should NEVER parse files.

It only:

* polls Graph delta
* downloads file
* creates metadata manifest
* sends parse job

`sync_daemon.py`

```python id="7v07tg"
from common.redis_queue import parse_queue, delete_queue
from graph_client import GraphClient
import uuid, os, time, json

gc = GraphClient()

while True:
    result = gc.delta_files(os.getenv("SHAREPOINT_DRIVE_ID"))

    for item in result.get("value", []):

        if "deleted" in item:
            delete_queue.enqueue("deletion_handler.process_delete", item["id"])
            continue

        if "file" not in item:
            continue

        local_name = f"/storage/incoming_docs/{uuid.uuid4()}_{item['name']}"
        gc.download_file(item["id"], local_name)

        manifest = {
            "sp_file_id": item["id"],
            "filename": item["name"],
            "sharepoint_url": item["webUrl"],
            "last_modified": item["lastModifiedDateTime"],
            "site_name": "Corporate",
            "library_name": "Documents",
            "permissions": []
        }

        parse_queue.enqueue("parse_consumer.process_parse", local_name, manifest)

    time.sleep(300)
```

---

# 4. Parser workers (horizontal scalable)

Now parser workers can scale independently:

* parser_worker_1
* parser_worker_2
* parser_worker_3

Useful because Docling is CPU heavy.

---

## parse_consumer.py

```python id="g78j1v"
from docling_parser import parse_document
from docling_chunker import build_chunk_payloads
from common.redis_queue import embed_queue

def process_parse(filepath, manifest):
    try:
        json_doc, markdown = parse_document(filepath)
        chunks = build_chunk_payloads(json_doc, manifest)
        embed_queue.enqueue("embed_consumer.process_embed", chunks)
    except Exception as e:
        print("PARSE FAILED", filepath, str(e))
```

---

# 5. Embedding workers

Again horizontally scalable.

Embedding worker only receives semantic chunks.

---

## embed_consumer.py

```python id="6wrdxg"
from qdrant_indexer import upsert_chunks

def process_embed(chunks):
    upsert_chunks(chunks)
```

---

# 6. Enterprise deletion handling (critical and often forgotten)

If SharePoint deletes a file:

its vectors MUST disappear.

Otherwise users retrieve ghost documents.

---

## deletion_handler.py

```python id="7sjql7"
from common.redis_queue import delete_queue

def process_delete(sp_file_id):
    delete_queue.enqueue("deletion_consumer.remove_vectors", sp_file_id)
```

---

## deletion_consumer.py

```python id="3rj7tm"
from qdrant_client import QdrantClient
from qdrant_client.http import models
import os

qdrant = QdrantClient(url=os.getenv("QDRANT_URL"))

def remove_vectors(sp_file_id):
    qdrant.delete(
        collection_name="sharepoint_knowledge",
        points_selector=models.FilterSelector(
            filter=models.Filter(
                must=[
                    models.FieldCondition(
                        key="doc_id",
                        match=models.MatchValue(value=sp_file_id)
                    )
                ]
            )
        )
    )
```

Now corpus stays truthful.

---

# 7. Qdrant collection schema (must be explicit)

Create once:

```python id="ukn5wf"
from qdrant_client import QdrantClient
from qdrant_client.http.models import Distance, VectorParams

qdrant.recreate_collection(
    collection_name="sharepoint_knowledge",
    vectors_config=VectorParams(size=3072, distance=Distance.COSINE)
)
```

3072 because text-embedding-3-large.

Payload indexes to add:

* doc_id
* site
* library
* modified
* permissions

for metadata filtering speed.

---

# 8. Add reranking (this matters a lot)

Pure vector top-k is mediocre.

We need:

* vector candidate top 25
* lexical overlap score
* heading boost
* freshness boost

Simple enterprise rerank:

```python id="4e3r3i"
def rerank(query, hits):
    scored = []

    for h in hits:
        score = h.score

        if query.lower() in h.payload.get("title","").lower():
            score += 0.15

        if query.lower() in h.payload.get("content","").lower():
            score += 0.08

        scored.append((score, h.payload))

    scored.sort(reverse=True, key=lambda x: x[0])
    return [x[1] for x in scored]
```

Simple but materially better.

Later you can swap with BGE reranker.

---

# 9. Citation builder (trust layer)

Users trust answers only when they can click source docs.

So RAG API should return:

```json id="76v2pc"
{
  "context": "...",
  "citations": [
    {
      "title": "Employee Handbook",
      "url": "https://tenant.sharepoint.com/...",
      "section": "Vacation Carryover"
    }
  ]
}
```

`citation_builder.py`

```python id="z2d1gd"
def build_citations(docs):
    cites = []
    for d in docs:
        cites.append({
            "title": d["title"],
            "url": d["sharepoint_url"],
            "section": d["title"]
        })
    return cites
```

---

# 10. RAG retrieve endpoint upgraded

Now `/retrieve` should return:

* top docs
* citations
* debug scores (optional admin mode)

This is useful for observability.

---

# 11. Open-WebUI pipeline upgraded with source forcing

This is important:

the pipeline should instruct LLM:

> answer only from supplied docs
> if uncertain say not found
> always cite source section titles

Without this, LLM hallucinates around context.

---

## upgraded prompt injection

```python id="8wq3w0"
body["messages"][-1]["content"] = f"""
You are answering from internal SharePoint enterprise knowledge.

Rules:
1. Use only supplied context.
2. If answer not present, say information not found in SharePoint knowledge.
3. Quote section titles when citing.
4. Do not invent policy details.

Context:
{context}

Question:
{query}
"""
```

This single prompt reduces hallucination substantially.

---

# 12. Docker compose enterprise additions

Now we need multiple workers.

```yaml id="4ck4mq"
parser_worker:
  build: ./services/parser_worker
  deploy:
    replicas: 3

indexer_worker:
  build: ./services/indexer_worker
  deploy:
    replicas: 2
```

Docling scaling is now isolated.

---

# 13. Enterprise observability (strongly recommended)

Add Postgres or SQLite log table:

store:

* query
* retrieved chunks
* final citations
* latency
* user feedback

Why?

Because eventually management asks:

> why did the bot answer this?

You need auditability.

---

# 14. Where most enterprise RAGs still fail after this

Even with all this, there are still **three advanced issues**:

### issue A — user identity passthrough from Open-WebUI SSO to ACL filter

### issue B — incremental delta token persistence and crash recovery

### issue C — Docling table/image enrichment for high fidelity retrieval

These are not optional in a real corporate deployment.

---

# Important note

We are now no longer discussing "how to make a chatbot."

We are designing:

> a full internal knowledge retrieval platform.

This is a medium-sized software project.

---

# Best next move

The next thing I should give you is the part engineers usually need most:

> the exact Open-WebUI integration method:
>
> * how to install this as a Pipeline
> * how Open-WebUI sends the prompt
> * how to expose citations in the chat
> * how to pass logged-in username/groups
> * how models are selected

and then after that:

> full Dockerfiles + requirements + install commands

Just say:

> continue with Open-WebUI integration

# Open WebUI integration
> continue with OpenWebUI integration

Excellent.

This is the **make-or-break layer**, because a lot of people build a good RAG backend and then integrate it poorly into Open-WebUI, which causes:

* context not injected correctly
* users bypassing RAG
* citations not shown
* no user identity passed
* prompts conflicting with model system prompts
* terrible admin maintainability

We want Open-WebUI to behave like:

> **a clean enterprise AI assistant whose primary knowledge source is SharePoint.**

Not just "ChatGPT with random attached docs."

---

# First: understand the 3 possible integration methods in Open-WebUI

Open-WebUI currently gives you three realistic enterprise integration patterns. Open-WebUI’s Pipelines framework is specifically intended for pre/post processing and external RAG augmentation between user chat and model execution.

---

## METHOD 1 — Native Knowledge Upload

Not suitable. Ignore.

Manual, weak, no enterprise orchestration.

---

## METHOD 2 — Tool / Function Calling

Model decides whether to call a retrieval tool.

Problem:

* model may or may not call it
* inconsistent grounding
* can answer from prior model knowledge instead
* hard to enforce

Good for optional search, bad for mandatory enterprise grounding.

---

## METHOD 3 — Pipeline Interceptor (**this is what we use**)

Pipeline intercepts every user message **before model sees it**.

Pipeline does:

1. capture user question
2. capture user identity
3. call your RAG API
4. inject retrieved context
5. rewrite final prompt
6. send to LLM

This guarantees grounding.

So:

> we are building a mandatory inlet pipeline.

---

# OPEN-WEBUI FLOW WE WANT

```text id="obdk8j"
User asks question
    ↓
Open-WebUI Pipeline inlet()
    ↓
Pipeline calls enterprise RAG API
    ↓
Gets top SharePoint chunks + citations
    ↓
Pipeline rewrites prompt with enterprise grounding
    ↓
Selected LLM answers
    ↓
Pipeline appends citations block
    ↓
User sees trusted response
```

---

# 1. Where this code lives in Open-WebUI

Open-WebUI Pipelines are usually mounted under the Pipelines container or imported into the backend pipeline folder, depending on deployment mode.

Your repo already has:

```text id="pplpm1"
services/openwebui_pipeline/sharepoint_rag_pipeline.py
```

This file becomes the enterprise interceptor.

---

# 2. The actual Pipeline class structure

Open-WebUI pipeline format expects a `Pipeline` class with hooks.

We care mainly about:

* `inlet()`  ← before LLM
* `outlet()` ← after LLM (for citations formatting)

---

## production version

```python id="gbw1xt"
import requests
import os
from typing import Optional

class Pipeline:

    def __init__(self):
        self.rag_url = os.getenv("RAG_API_URL")

    async def inlet(self, body: dict, user: Optional[dict] = None) -> dict:
        query = body["messages"][-1]["content"]

        user_email = None
        user_groups = []

        if user:
            user_email = user.get("email")
            user_groups = user.get("groups", [])

        rag = requests.post(
            f"{self.rag_url}/retrieve",
            json={
                "query": query,
                "user_email": user_email,
                "user_groups": user_groups
            },
            timeout=20
        ).json()

        docs = rag.get("documents", [])
        citations = rag.get("citations", [])

        context = "\n\n".join([
            f"[SECTION: {d['title']}]\n{d['content']}" for d in docs
        ])

        grounded_prompt = f'''
You are the company's internal SharePoint knowledge assistant.

STRICT RULES:
- Answer ONLY from supplied SharePoint context.
- If information is absent, say "I could not find that in SharePoint knowledge."
- Never use outside/general model knowledge for company policies.
- Prefer concise factual answers.
- Mention section titles when relevant.

SHAREPOINT CONTEXT:
{context}

USER QUESTION:
{query}
'''

        body["messages"][-1]["content"] = grounded_prompt
        body["__citations__"] = citations

        return body
```

This forces every prompt through SharePoint retrieval.

---

# 3. Why `body["__citations__"]` matters

We need to preserve citations after model response.

Because Open-WebUI normally sends only model text back.

We store citations in body metadata so `outlet()` can append them.

---

# 4. Outlet() — append clickable source references

```python id="b4c91g"
    async def outlet(self, body: dict, user: Optional[dict] = None) -> dict:
        citations = body.get("__citations__", [])

        if citations:
            citation_text = "\n\n---\n**Sources:**\n"
            for c in citations:
                citation_text += f"- {c['title']} ({c['url']})\n"

            if "messages" in body and len(body["messages"]) > 0:
                body["messages"][-1]["content"] += citation_text

        return body
```

Now every answer visibly shows SharePoint source links.

Users love this because:

> trust instantly increases.

---

# 5. Very important: enforce RAG only on selected models

You probably do NOT want every model in Open-WebUI to be SharePoint grounded.

You may have:

* General GPT-4.1
* Coding assistant
* Research model
* SharePoint Enterprise Assistant

Only one should use this pipeline.

Open-WebUI allows assigning pipelines to model presets / routed models in admin configuration.

Create model:

```text id="0s2rmq"
Corporate Knowledge Assistant
```

Attached to this pipeline.

This gives users a dedicated internal assistant.

Much cleaner than hijacking all chats.

---

# 6. Pass user identity from Open-WebUI login

This is enterprise critical.

If Open-WebUI is configured with SSO / OIDC / Azure Entra:

logged-in user metadata can be made available to pipeline hooks depending on auth setup and headers/session mapping.

We need:

* email
* username
* groups/roles

Why?

Because your RAG API ACL filter must know what user can access.

Pipeline sends:

```json id="pv6h77"
{
   "query": "...",
   "user_email": "john@company.com",
   "user_groups": ["HR","Managers"]
}
```

Then backend filters unauthorized chunks.

---

# 7. RAG API response format (finalized contract)

Your backend `/retrieve` should return:

```json id="5l8h2z"
{
  "documents": [
    {
      "title": "Vacation Carryover Policy",
      "content": "...",
      "sharepoint_url": "https://..."
    }
  ],
  "citations": [
    {
      "title": "Vacation Carryover Policy",
      "url": "https://..."
    }
  ]
}
```

Pipeline depends on this contract.

---

# 8. Important latency guardrail

Do not let RAG retrieval make Open-WebUI feel slow.

Target:

* retrieval < 2 sec
* total prompt injection < 3 sec

Pipeline timeout should degrade gracefully.

Add fallback:

```python id="u7k0jg"
try:
    rag = requests.post(...).json()
except:
    return body
```

If backend temporarily fails, chat still works.

---

# 9. Strong recommendation: show "no source found" honestly

If RAG API returns no docs:

inject:

```text id="oh9kl8"
No relevant SharePoint context was found for this question.
```

and instruct model to state that.

This is far better than hallucinated answers.

---

# 10. Admin installation inside Open-WebUI

Deployment options:

### Option A — Pipelines container (recommended)

Open-WebUI official Pipelines sidecar service

### Option B — mount custom pipeline file in backend

Pipelines sidecar is cleaner for upgrades and separation.

---

# 11. What users will experience

User opens Open-WebUI and chooses:

> Corporate Knowledge Assistant

Then asks:

> "How many PTO days can be carried over?"

System silently:

* searches SharePoint
* injects retrieved sections
* model answers
* cites source links

Feels native.

That is exactly what we want.

---

# 12. There is one final huge engineering area left

We have now built:

* ingestion
* parsing
* indexing
* retrieval
* Open-WebUI integration

But we still have not covered:

> **deployment artifacts**

meaning:

* Dockerfiles
* requirements.txt
* exact install commands
* Open-WebUI pipeline registration
* Qdrant init script
* Azure App registration settings
* production startup sequence

Without those, engineers still have to improvise.

So the next step is the full DevOps package.

Just say:

> continue with deployment package

# Deployment package

Perfect.

Now we finish this into something your engineers can actually stand up.

This section gives you the **deployment package**:

> Dockerfiles
> requirements
> service startup
> Qdrant init
> Azure registration
> Open-WebUI mounting
> production boot sequence

After this, you'll have essentially the first full implementation draft.

---

# 1. Final deployment topology

We are deploying these containers:

```text id="1sp4qn"
open-webui
open-webui-pipelines
rag-api
sharepoint-worker
parser-worker (xN)
indexer-worker (xN)
redis
qdrant
```

This keeps Open-WebUI isolated from backend compute workers.

---

# 2. Docker Compose (production version)

```yaml id="95dsv4"
version: '3.9'

services:

  redis:
    image: redis:7
    restart: always

  qdrant:
    image: qdrant/qdrant:latest
    restart: always
    ports:
      - "6333:6333"
    volumes:
      - ./volumes/qdrant:/qdrant/storage

  rag-api:
    build:
      context: .
      dockerfile: ./services/rag_api/Dockerfile
    env_file: .env
    restart: always
    depends_on:
      - qdrant
      - redis
    ports:
      - "8000:8000"

  sharepoint-worker:
    build:
      context: .
      dockerfile: ./services/sharepoint_worker/Dockerfile
    env_file: .env
    restart: always
    depends_on:
      - redis

  parser-worker:
    build:
      context: .
      dockerfile: ./services/parser_worker/Dockerfile
    env_file: .env
    restart: always
    depends_on:
      - redis
    deploy:
      replicas: 3

  indexer-worker:
    build:
      context: .
      dockerfile: ./services/indexer_worker/Dockerfile
    env_file: .env
    restart: always
    depends_on:
      - redis
      - qdrant
    deploy:
      replicas: 2

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    restart: always
    ports:
      - "3000:8080"
    volumes:
      - openwebui_data:/app/backend/data

  open-webui-pipelines:
    image: ghcr.io/open-webui/pipelines:main
    restart: always
    env_file: .env
    volumes:
      - ./services/openwebui_pipeline:/app/pipelines

volumes:
  openwebui_data:
```

---

# 3. Requirements files

---

## requirements/workers.txt

```txt id="7y4x6r"
docling
redis
rq
requests
msal
openai
qdrant-client
python-dotenv
```

---

## requirements/rag_api.txt

```txt id="8j3epd"
fastapi
uvicorn
openai
qdrant-client
python-dotenv
requests
```

---

## requirements/pipeline.txt

```txt id="1wmg9m"
requests
```

---

# 4. Dockerfiles

---

## services/rag_api/Dockerfile

```dockerfile id="5ewgsa"
FROM python:3.11-slim

WORKDIR /app
COPY requirements/rag_api.txt .
RUN pip install --no-cache-dir -r rag_api.txt

COPY services/rag_api /app

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## services/sharepoint_worker/Dockerfile

```dockerfile id="zdu1wo"
FROM python:3.11-slim

WORKDIR /app
COPY requirements/workers.txt .
RUN pip install --no-cache-dir -r workers.txt

COPY services/sharepoint_worker /app
COPY services/common /app/common

CMD ["python", "sync_daemon.py"]
```

---

## services/parser_worker/Dockerfile

```dockerfile id="7w65is"
FROM python:3.11-slim

WORKDIR /app
COPY requirements/workers.txt .
RUN pip install --no-cache-dir -r workers.txt

COPY services/parser_worker /app
COPY services/common /app/common

CMD ["rq", "worker", "parse_queue", "--with-scheduler"]
```

---

## services/indexer_worker/Dockerfile

```dockerfile id="4c1qos"
FROM python:3.11-slim

WORKDIR /app
COPY requirements/workers.txt .
RUN pip install --no-cache-dir -r workers.txt

COPY services/indexer_worker /app
COPY services/common /app/common

CMD ["rq", "worker", "embed_queue", "delete_queue", "--with-scheduler"]
```

---

# 5. Qdrant initialization script

Need to create collection once on startup.

`services/indexer_worker/init_qdrant.py`

```python id="f62xhq"
from qdrant_client import QdrantClient
from qdrant_client.http.models import Distance, VectorParams
import os

qdrant = QdrantClient(url=os.getenv("QDRANT_URL"))

collections = [c.name for c in qdrant.get_collections().collections]

if "sharepoint_knowledge" not in collections:
    qdrant.recreate_collection(
        collection_name="sharepoint_knowledge",
        vectors_config=VectorParams(size=3072, distance=Distance.COSINE)
    )
```

Run this once before workers start.

---

# 6. Azure App Registration (SharePoint access)

In Microsoft Azure / Microsoft Entra ID:

Create:

> App registrations → New registration

Add API permissions:

* Microsoft Graph → Application Permissions

  * Sites.Read.All
  * Files.Read.All

Grant admin consent.

Create:

* client id
* tenant id
* client secret

Put into `.env`

This is the service principal that reads SharePoint.

---

# 7. Environment file final

```bash id="12g7qn"
GRAPH_TENANT_ID=
GRAPH_CLIENT_ID=
GRAPH_CLIENT_SECRET=
SHAREPOINT_DRIVE_ID=

OPENAI_API_KEY=

QDRANT_URL=http://qdrant:6333
REDIS_HOST=redis
RAG_API_URL=http://rag-api:8000
```

---

# 8. Open-WebUI Pipelines registration

Open-WebUI Pipelines service auto-loads `.py` pipeline files from mounted `/app/pipelines`. Official pipeline deployment docs describe this mount-and-load behavior.

So your file:

```text id="qaqo2s"
services/openwebui_pipeline/sharepoint_rag_pipeline.py
```

gets automatically discovered.

Then in Open-WebUI Admin:

* Connections
* Pipelines
* Enable SharePoint RAG Pipeline

Then attach it to model:

> Corporate Knowledge Assistant

---

# 9. Production startup sequence

This matters a lot.

Bring system up in this order:

```bash id="w4k30u"
docker compose up redis qdrant -d
python services/indexer_worker/init_qdrant.py
docker compose up rag-api -d
docker compose up parser-worker indexer-worker sharepoint-worker -d
docker compose up open-webui open-webui-pipelines -d
```

Why?

Because:

* Qdrant must exist before indexing
* RAG API before Open-WebUI pipeline calls it
* workers after queues available

---

# 10. First indexing cycle expectation

On first boot:

sharepoint-worker begins polling.

Pipeline:

* download docs
* enqueue parse
* Docling parse
* chunk
* embed
* Qdrant upsert

Large SharePoint libraries may take hours first run.

That is normal.

Subsequent runs are delta only.

---

# 11. Recommended production enhancements before users touch it

There are 5 things I would strongly add before launch:

### A. persistent delta token storage in Redis/Postgres

for crash-safe continuation

### B. failed job dead letter queue

for bad documents

### C. query logging DB

for observability

### D. admin dashboard `/health`

to inspect queue depth/index counts

### E. model answer feedback buttons

for RAG quality tuning

These are the operational maturity features.

---

# 12. Honest engineering assessment

At this point:

> you now have the full enterprise deployment package for version 1.

Meaning:

* architecture complete
* code skeleton complete
* Open-WebUI complete
* DevOps complete

But…

there is still one thing that determines whether users say:

> "wow this is useful"

or

> "this bot is dumb"

That thing is:

# retrieval quality engineering

because production RAG success is 70% retrieval quality, not infrastructure.

We still have not tuned:

* Docling chunk semantics
* hybrid BM25 + vector
* reranking
* query rewriting
* conversational memory retrieval
* metadata narrowing
* table-aware retrieval

That is the intelligence layer.

And this layer matters massively.

---

## My recommendation:

Before you build, let me give you the **RAG quality optimization layer** — the part that makes this system actually answer well.

Just say:

> continue with retrieval optimization
Excellent.

This is the part that separates:

> “we built a SharePoint chatbot”

from

> “we built something employees will actually trust and use.”

Because here is the uncomfortable truth:

> most enterprise RAG systems fail **after successful deployment** because retrieval quality is mediocre.

Infrastructure works.
Users ask questions.
Bot returns vaguely related chunks.
LLM produces half-right answers.
Trust dies.

So now we engineer the **retrieval intelligence layer**.

---

# ENTERPRISE RETRIEVAL OPTIMIZATION STACK

We are going to optimize 6 major areas:

```text id="4awz9o"
1. Better chunk semantics
2. Metadata-aware narrowing
3. Hybrid retrieval
4. Query rewriting
5. Reranking
6. Conversational retrieval memory
```

Each one materially improves answer quality.

---

# 1. Better chunk semantics (Docling-specific and VERY important)

Your current chunker is still too primitive:

> heading + body text

That works, but it wastes a lot of Docling’s document intelligence.

Docling gives you:

* hierarchy
* table blocks
* captions
* lists
* page anchors
* parent/child nodes
* semantic labels

We should exploit this.

---

## What bad chunking does

Suppose a SharePoint HR handbook contains:

```text id="oqb3oj"
Section: Paid Leave
Subsection: Vacation Carryover
Employees may carry over 5 unused vacation days...

Subsection: Sick Leave
Unused sick leave does not...
```

Naive chunking often merges these into one blob.

Then retrieval returns mixed policy sections.

LLM confuses them.

---

## What enterprise chunking should do

Every chunk should become a **retrieval unit with semantic identity**:

```python id="x0fcdu"
{
   "doc_id": "...",
   "section_path": "Paid Leave > Vacation Carryover",
   "heading": "Vacation Carryover",
   "parent_heading": "Paid Leave",
   "content": "...",
   "keywords": ["vacation", "carry over", "unused days"],
   "table_present": False,
   "page_refs": [14]
}
```

Now retrieval can score:

* heading match
* parent heading match
* keyword overlap

instead of just embedding similarity.

This is huge.

---

## Upgrade Docling chunker to hierarchical path chunking

Instead of one current heading, maintain heading stack:

```python id="zhf1x0"
heading_stack = ["Paid Leave", "Vacation Carryover"]
section_path = " > ".join(heading_stack)
```

That string becomes highly valuable retrieval metadata.

---

# 2. Metadata-aware narrowing (extremely underrated)

Most RAG systems search the entire company corpus for every question.

This is dumb.

Because enterprise questions often imply domain.

Question:

> “what is the PTO carryover limit?”

almost certainly HR.

Question:

> “how do I submit CAPEX approval?”

almost certainly Finance or Procurement.

So before vector search:

> infer probable business domain.

---

## Domain classifier layer

Simple first-pass keyword classifier:

```python id="b31h2o"
DOMAIN_HINTS = {
    "hr": ["pto", "vacation", "leave", "holiday", "employee", "benefits"],
    "finance": ["invoice", "capex", "expense", "budget", "approval"],
    "it": ["vpn", "laptop", "password", "access", "software"]
}
```

Then narrow Qdrant filter:

```python id="8e47i4"
site == "HR Portal"
```

or

```python id="8fdvsa"
library == "Finance Docs"
```

This dramatically improves relevance because vector search corpus shrinks.

---

# 3. Hybrid retrieval (mandatory)

Pure embeddings are not enough.

Why?

Enterprise docs contain:

* acronyms
* exact policy terms
* form names
* ticket IDs
* product codes
* abbreviations

Embeddings often miss exact lexical anchors.

Example:

> “CAPEX-17 form”

Vector search alone often fails.

Need:

> vector retrieval + lexical retrieval + merge.

---

## Recommended stack

### Vector:

Qdrant semantic top 20

### Lexical:

BM25 / Tantivy / Whoosh / Elasticsearch top 20

Then union results.

---

## Why this matters

Semantic catches:

* “vacation rollover” ≈ “carry over unused PTO”

Lexical catches:

* “FORM-19B”
* “VPN”
* “SAP-AP invoice”

Together = enterprise robustness.

---

# 4. Query rewriting (massive quality booster)

Employees ask bad questions.

Examples:

* “carryover pto?”
* “laptop broken who ask”
* “maternity rules switzerland office”

Raw user text is often not ideal retrieval text.

So before retrieval:

LLM micro-rewrite query into search-friendly enterprise language.

---

## Query rewrite prompt

```python id="9yxx8m"
Rewrite the employee question into a precise enterprise document search query.
Expand abbreviations where possible.
Infer likely policy terminology.
Return only rewritten query.
```

Input:

> "carryover pto?"

Output:

> "PTO vacation carryover unused paid leave policy"

This often gives a huge retrieval lift.

Use cheap small LLM for this.

---

# 5. Enterprise reranking (critical)

Now we have:

* vector hits
* BM25 hits
* metadata filtered hits

But top results are still noisy.

Need reranking.

---

## Multi-factor rerank score

```python id="1uhy5s"
final_score =
    semantic_score * 0.45 +
    lexical_score * 0.20 +
    heading_match * 0.15 +
    section_path_match * 0.10 +
    freshness_score * 0.05 +
    table_boost * 0.05
```

This is much stronger than raw cosine similarity.

---

## heading boost example

If query contains:

> carryover

and chunk heading is:

> Vacation Carryover

boost heavily.

Users usually ask by section concept.

---

# 6. Conversational retrieval memory

This is one many systems miss.

Users ask:

> "How much PTO can I carry over?"
> then
> "does that apply to contractors too?"

Second question is useless standalone.

Pipeline must maintain prior retrieval intent.

So retrieve on:

```text id="r4jxqw"
Current question + previous user question + previous retrieved section headings
```

This preserves conversational continuity.

Open-WebUI sends prior messages already, so pipeline can construct retrieval query history.

---

# 7. Table-aware retrieval (Docling superpower)

SharePoint docs often contain policy tables:

| Region | PTO Carryover |
| ------ | ------------- |
| EU     | 5 days        |
| US     | 10 days       |

Normal embeddings underweight table rows.

We should render tables into sentence-enriched text too:

```text id="j0o9jm"
Table data:
Region EU PTO Carryover 5 days
Region US PTO Carryover 10 days
```

Store alongside markdown table.

This makes numerical policy retrieval far better.

---

# 8. Citation precision upgrade

Do not cite only document title.

Cite:

* document
* section path
* optional page

Example:

```text id="2eu5eg"
Source: Employee Handbook → Paid Leave → Vacation Carryover (Page 14)
```

This makes employees believe the system.

Trust is a UX feature.

---

# 9. No-answer confidence threshold

One of the biggest enterprise mistakes:

always answering something.

Need confidence gate:

if top reranked score < threshold:

> return "I could not find a reliable SharePoint source for that."

This is far superior to weak guesses.

---

# 10. The retrieval engine is now changing shape

Your RAG API is no longer just:

> embed query → Qdrant top5

It becomes:

```text id="ec4m8d"
user question
   ↓
query rewrite
   ↓
domain inference
   ↓
vector search
   ↓
BM25 lexical search
   ↓
merge
   ↓
ACL filter
   ↓
rerank
   ↓
confidence threshold
   ↓
return context + precise citations
```

This is enterprise-grade retrieval.

---

# Honest statement

Now we are entering the territory where this project becomes:

> a real internal search product

not merely “RAG around docs.”

And this is exactly where successful deployments win.

---

# Important engineering note

There are still two huge advanced subjects left that will matter in real deployment:

### A — SharePoint permissions/ACL sync from Microsoft Graph groups

(actual security trimming, not placeholder permissions=[])

### B — observability/admin dashboard/testing framework

(how you know retrieval is improving or failing)

These are both major.

I strongly recommend we do ACL sync next, because without security trimming many companies will refuse deployment.

Just say:

> continue with ACL/security model
Excellent.

This is one of the most underestimated parts of enterprise RAG:

> people build the chatbot first and think about security later.

That usually ends with:

> “legal says we cannot release this.”

Because if an employee can ask:

> “show me executive compensation policy drafts”

and retrieve unauthorized SharePoint files…

the project is dead.

So now we build:

> **true SharePoint security trimming / ACL-aware retrieval**

not fake placeholder permissions arrays.

---

# FIRST: understand the actual security problem

Your RAG index is a giant flattened copy of SharePoint content.

The moment content is copied into Qdrant:

> SharePoint's native permission enforcement is gone.

Qdrant does not know:

* SharePoint site permissions
* library permissions
* folder permissions
* file-level unique permissions
* Azure AD group membership

So **you must re-create access control metadata inside the index.**

This is called:

> security trimming.

Enterprise search systems like Microsoft Search and Elastic do this for exactly the same reason.

---

# WHAT ACL DATA MUST BE CAPTURED DURING INGESTION

Every SharePoint file must carry:

```python id="8n3m2h"
{
   "sp_file_id": "...",
   "sharepoint_url": "...",
   "allowed_users": [...],
   "allowed_groups": [...],
   "inherited_from": "...",
   "is_public_internal": False
}
```

Not optional.

Without this, retrieval cannot filter.

---

# 1. How to get permissions from SharePoint properly

This comes from Microsoft Graph permissions endpoints.

For each file item:

```http id="84h2bq"
/drives/{drive-id}/items/{item-id}/permissions
```

and sometimes inherited site/group membership from:

```http id="o9jzq0"
/sites/{site-id}/permissions
```

Graph exposes granted principals such as:

* user principals
* Azure AD groups
* SharePoint groups
* links (ignore anonymous links internally)

Microsoft documents these permission resources through Graph DriveItem permission APIs.

---

## Extend graph_client.py

```python id="jlbr8f"
def get_item_permissions(self, item_id):
    url = f"https://graph.microsoft.com/v1.0/drives/{os.getenv('SHAREPOINT_DRIVE_ID')}/items/{item_id}/permissions"
    return requests.get(url, headers=self.headers).json()
```

---

# 2. Normalize permissions into enterprise-friendly ACL manifests

Graph raw permissions are messy.

We normalize into:

```python id="q1f7l8"
{
   "allowed_users": [
       "john@company.com",
       "mary@company.com"
   ],
   "allowed_groups": [
       "HR",
       "Finance Managers",
       "Leadership Team"
   ],
   "is_public_internal": False
}
```

Need a parser:

```python id="r1u7me"
def normalize_permissions(graph_permissions):
    users = set()
    groups = set()

    for perm in graph_permissions.get("value", []):
        granted = perm.get("grantedToV2", {})

        user = granted.get("user")
        group = granted.get("group")

        if user and "email" in user:
            users.add(user["email"].lower())

        if group and "displayName" in group:
            groups.add(group["displayName"])

    return {
        "allowed_users": list(users),
        "allowed_groups": list(groups),
        "is_public_internal": len(users) == 0 and len(groups) == 0
    }
```

Now ACL metadata is deterministic.

---

# 3. Attach ACLs to every chunk payload

When parser builds chunk payloads:

```python id="4ej6gz"
payload = {
   ...
   "allowed_users": manifest["allowed_users"],
   "allowed_groups": manifest["allowed_groups"],
   "is_public_internal": manifest["is_public_internal"]
}
```

Every chunk inherits file permissions.

Important:

if folder/file has unique permissions, these must override parent inheritance.

---

# 4. User identity passthrough from Open-WebUI

Now Open-WebUI login identity matters.

You should configure Open-WebUI with Microsoft Entra ID / OIDC SSO so each logged-in employee has:

* email
* display name
* group claims (or resolvable groups)

Open-WebUI supports external auth providers and OIDC-based SSO setups that can expose authenticated user metadata.

Pipeline sends:

```json id="5clvzm"
{
   "query": "...",
   "user_email": "john@company.com",
   "user_groups": ["HR", "Managers"]
}
```

to RAG API.

---

# 5. Retrieval-time ACL filter (the hard gate)

Even if vector search returns perfect matches:

> user must not see unauthorized chunks.

So after retrieval candidate merge, before rerank finalization:

```python id="iz9k6h"
def acl_filter(hit, user_email, user_groups):
    payload = hit.payload

    if payload.get("is_public_internal"):
        return True

    if user_email in payload.get("allowed_users", []):
        return True

    if set(user_groups).intersection(set(payload.get("allowed_groups", []))):
        return True

    return False
```

Then:

```python id="96b4p8"
authorized_hits = [h for h in merged_hits if acl_filter(h, user_email, user_groups)]
```

This is the enterprise security gate.

---

# 6. Important: never leak unauthorized citations either

Very common mistake:

developers ACL filter context text but still leave citations metadata from unauthorized docs.

Do not do that.

Citations must be built **after ACL filter only**.

---

# 7. Group membership freshness problem (very important)

Employees change departments.

If user group claims are stale, access leaks or false denials happen.

Recommended:

Do not trust static group strings from old sessions.

Either:

### Option A — Open-WebUI SSO passes fresh group claims every login

or

### Option B — RAG API calls Graph `/memberOf` on demand and caches for 15 min

This is safer.

---

## Example Graph group lookup

```http id="ih0jg1"
/users/{user-id}/memberOf
```

Use to resolve:

* HR
* Finance
* Legal Restricted
* Executive Team

---

# 8. Public vs restricted corpus segmentation (smart enterprise pattern)

Do not put everything in one giant unrestricted collection.

Better:

```text id="6sj1ha"
sharepoint_public
sharepoint_restricted
sharepoint_exec
```

Then route search:

* public first
* restricted if user has groups
* exec only if eligible

This reduces accidental leakage risk and improves search relevance.

---

# 9. Logging for security audit

Every retrieval should log:

```python id="d1x44u"
{
   "user_email": "...",
   "query": "...",
   "returned_doc_ids": [...],
   "blocked_doc_ids": [...],
   "timestamp": ...
}
```

Why?

Because security/legal may ask:

> did anyone query sensitive HR documents?

Need audit trail.

---

# 10. Sensitive document no-answer policy

For certain libraries:

* Legal
* Executive
* M&A
* Payroll

even authorized retrieval may require stricter answer behavior.

You can flag:

```python id="t5x20w"
sensitivity = "high"
```

and force:

* no summarization
* only direct citations
* or deny AI synthesis

This is optional but common in regulated companies.

---

# 11. Security model summary (this is now real enterprise safe)

Final retrieval path becomes:

```text id="9c5w7o"
User query
  ↓
Query rewrite
  ↓
Domain narrow
  ↓
Vector + BM25 retrieval
  ↓
ACL authorization filter
  ↓
Rerank authorized hits only
  ↓
Confidence threshold
  ↓
Return context + authorized citations only
```

Now unauthorized SharePoint knowledge is invisible.

That is what legal/security teams need to hear.

---

# Honest assessment

At this point we have now designed:

* ingestion
* parsing
* indexing
* retrieval optimization
* Open-WebUI integration
* deployment
* security trimming

This is already the skeleton of a serious internal enterprise knowledge assistant.

---

# There is now one final major engineering layer that determines long-term success:

> observability, testing, and admin analytics

Because once this is live, management will ask:

* Is it accurate?
* What are users asking?
* Which docs fail retrieval?
* Which answers get thumbs down?
* Which libraries are low quality?

Without measurement, RAG tuning becomes guesswork.

So the final major chapter is:

> monitoring + evaluation + admin dashboard + feedback loops

Just say:

> continue with observability and evaluation
