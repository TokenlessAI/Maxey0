# Maxey0 — Agentic Memory & Modular AI Orchestration

Maxey0 is a **agentic memory platform** and **modular AI orchestration framework** for multi‑agent systems, dynamic memory, and real‑time ingestion. It stitches together **Neo4j** (graph memory & provenance), **vector backends** (Redis/FAISS), and **tool‑augmented agents** behind clean FastAPI services.

---

## Core Principles
- **Modularity first** — swappable components, clear boundaries
- **Stateful agents** — persistent, queryable memory (graph + vectors)
- **Multi‑modal context** — text, structured data, OCR and more
- **Continuous learning** — streaming ingestion and provenance
- **Seamless orchestration** — RAG, MCP, LangChain/CrewAI adapters

---

## System Overview
- **Two‑plane memory**
  - **Graph plane (Neo4j)**: provenance, topics, relationships (e.g., `:Agent`, `:Memory`, `:Topic`, `[:WROTE]`, `[:TAGGED]`)
  - **Vector plane (Redis/FAISS)**: fast ANN query by scope
- **Streaming/A2A**: Redis Streams (consumer groups, ACK, DLQ, audit)
- **Optional GNN router**: GraphSAGE/GIN for semantic routing
- **Tool suite**: web server, code interpreter sandbox, virtual env manager, web search, analytics dashboard
- **Observability**: DLQ pressure, stream rates, graph/redis health
- **Magnitude Spectrum mapping**: deterministic, model‑agnostic embeddings

---

## Repository Layout (authoritative)
```
Maxey0/
├── README.md
├── requirements.txt
├── pyproject.toml
├── LICENSE
├── .gitignore
├── docker/
│ ├── compose.full.yml
│ ├── Dockerfile.api
│ ├── Dockerfile.runner
│ ├── Dockerfile.gnn
│ ├── Dockerfile.web
│ └── init-neo4j.cypher
├── configs/
│ ├── settings.toml
│ ├── logging.yaml
│ └── secrets.env.example
├── data/
│ ├── vectors/
│ │ └── example_vectors.json
│ ├── conversations/
│ │ ├── maxey0_conversations.json
│ │ └── maxey00_conversations.json
│ ├── preembedded/
│ │ ├── maxey0_vectors.jsonl
│ │ └── maxey00_vectors.jsonl
│ └── agents/
│ ├── maxey_agents.cypher
│ └── agents_metadata.json
│ └── seed_120_agents.cypher
├── docs/
│ ├── architecture.md
│ ├── agent_lifecycle.md
│ ├── api_reference.md
│ └── developer_guide.md
├── scripts/
│ ├── ingest_knowledge.py
│ ├── ingest_conversations.py
│ ├── watch_ingest.py
│ └── no_placeholder_guard.py
├── Maxey0/
│ ├── init.py
│ ├── superagent.py
│ ├── core/
│ │ ├── config.py
│ │ ├── logging.py
│ │ └── utils.py
│ ├── api/
│ │ ├── init.py
│ │ ├── memory_api.py [✅ PRINTED]
│ │ ├── runner_api.py
│ │ └── mcp_api.py
│ ├── server/
│ │ ├── init.py
│ │ ├── endpoints_ops.py
│ │ ├── retriever.py
│ │ ├── runner.py
│ │ ├── a2a_streams.py
│ │ └── mcp_server.py
│ ├── a2a/
│ │ ├── init.py
│ │ ├── streams.py
│ │ ├── memory_sharing.py
│ │ └── cross_framework_adapter.py
│ ├── mms/
│ │ ├── init.py
│ │ └── memory_manager.py
│ ├── latent/
│ │ ├── init.py
│ │ ├── latent_space_map.py
│ │ └── magnitude_spectrum.py
│ ├── vectors/
│ │ ├── init.py
│ │ └── backends/
│ │ ├── init.py
│ │ ├── memory.py
│ │ ├── redis_scan.py
│ │ └── faiss_scan.py
│ ├── tools/
│ │ ├── init.py
│ │ ├── web_server_tool.py
│ │ ├── code_interpreter_tool.py
│ │ ├── virtual_env_tool.py
│ │ ├── web_search_tool.py
│ │ ├── analytics_dashboard_tool.py
│ │ ├── ocr_tool.py
│ │ ├── pairwise_distance_tool.py
│ │ └── sandbox_tool.py
│ ├── adapters/
│ │ ├── init.py
│ │ ├── langchain_adapter.py
│ │ ├── crewai_adapter.py
│ │ ├── nvidia_agent_adapter.py
│ │ └── swarm_adapter.py
│ ├── ops/
│ │ ├── init.py
│ │ ├── drift_analytics.py
│ │ ├── observability.py
│ │ └── metrics_collector.py
│ └── neo4j/
│ ├── init.py
│ └── neo4j_client.py
└── tests/
├── init.py
├── test_memory_api.py
├── test_runner_api.py
├── test_a2a.py
├── test_latent_space_map.py
├── test_magnitude_spectrum.py
├── test_tools.py
└── test_gnn_router.py
```

> The layout above is **canonical** for Maxey0 and originates from `readme1.txt`.

---

## Prerequisites
- Python **3.11+** (3.12 recommended)
- **Neo4j 5+** (local or Aura) with Bolt enabled
- **Redis 6+** (with vector module) *or* FAISS for local ANN
- Docker & Compose (optional, recommended)
- PyTorch + PyG (optional, for GNN router experimentation)

---

## Installation

### Option A — uv (fast & reproducible)
```bash
git clone https://github.com/TokenlessAI/Maxey0.git
cd Maxey0
uv pip install -r requirements.txt
```

### Option B — venv + pip
```bash
git clone https://github.com/TokenlessAI/Maxey0.git
cd Maxey0
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

---

## Configuration

Create your environment file and adjust as needed:
```bash
cp configs/secrets.env.example configs/secrets.env
```

Minimal variables in `configs/secrets.env`:
```env
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=changeme
NEO4J_DATABASE=neo4j

REDIS_URL=redis://localhost:6379/0

# Optional
A2A_STREAM_NAMESPACE=m0
BRAVE_SEARCH_API_KEY=
```

Initialize Neo4j constraints/indexes (first run only if not using Compose bootstrap):
```bash
cypher-shell -a bolt://localhost:7687 -u "$NEO4J_USER" -p "$NEO4J_PASSWORD" < docker/init-neo4j.cypher
```

---

## Run with Docker Compose

Bring up Neo4j, Redis, and APIs:
```bash
docker compose -f docker/compose.full.yml up -d
```

Expected services (when started):
- Memory API → `http://localhost:18080`
- Runner API → `http://localhost:18081`
- Analytics Dashboard Tool → `http://localhost:8090`
- Web Server Tool → `http://localhost:8080`

Shut down:
```bash
docker compose -f docker/compose.full.yml down -v
```

---

## Start Services Manually

```bash
# Memory API
uvicorn Maxey0.api.memory_api:app --host 0.0.0.0 --port 18080

# Runner API
uvicorn Maxey0.api.runner_api:app --host 0.0.0.0 --port 18081
```

---

## Core APIs (selected)

**Memory API** (`http://localhost:18080`)
- `POST /memory/insert` — insert vectors
- `POST /memory/insert_text` — insert text (auto‑embed)
- `POST /memory/query` — KNN search + optional provenance hydration
- `POST /memory/meta` — patch metadata
- `POST /memory/delete` — delete entries

**Runner API** (`http://localhost:18081`)
- `POST /run` — execute a tool module; supports multi‑step plans and Neo4j audit

---

## Tools (selected)
- **Web Server Tool** — serve HITL workspaces, artifacts, logs
- **Code Interpreter Tool** — sandboxed Python; optional ephemeral venv
- **Virtual Env Tool** — create/install/info for agent venvs
- **Web Search Tool** — pluggable search providers
- **Analytics Dashboard Tool** — live metrics via Redis/Neo4j
- **Agent Development Kit** — define/register/spawn agents dynamically

All tools expose a single `run(**kwargs) -> dict` entrypoint.

---

## Latent Space Mapping

Default embeddings use the **Magnitude Spectrum Generator** (`Maxey0/latent/magnitude_spectrum.py`) for deterministic, robust, dimension‑tunable vectors. Access via `LatentSpaceMapper` (`latent_space_map.py`):
```python
from Maxey0.latent.latent_space_map import LatentSpaceMapper
emb = LatentSpaceMapper().embed("hello world")
```

---

## Neo4j Graph Schema (provenance)

- Unique constraints: `(:Memory {id})`, `(:Agent {name})`, `(:Topic {name})`
- Core patterns:
  - `(:Agent {…})-[:WROTE]->(:Memory {…})`
  - `(:Memory)-[:TAGGED]->(:Topic {name})`

When `with_provenance=true`, queries auto‑hydrate related nodes/edges.

---

## A2A Messaging (Redis Streams)
- Main stream: `${A2A_STREAM_NAMESPACE}:a2a` (default `m0:a2a`)
- DLQ: `${NS}:dlq`
- Audit: `${NS}:audit`

Consumer groups with ACK & retry; failures route to DLQ. Dashboard computes backlog & rates.

---

## Testing
```bash
pytest -q
# Optional guard to enforce “no placeholders”:
python scripts/no_placeholder_guard.py Maxey0
```

---

## CI (recommended GitHub Actions)
- **CI (lint + tests via uv)** — ruff, mypy, pytest
- **Integration (Neo4j service)** — spin up `neo4j:5` service container
- **Docker publish** — build & push `api`, `runner`, `web` to GHCR
- **CodeQL** — security scanning (v3)
- **Container hygiene** — Hadolint + Trivy (upload SARIF)

Starter workflow files live under `.github/workflows/`.

---

## License

This project is licensed under the **Apache License, Version 2.0**.

SPDX identifier to add to source files:
```
SPDX-License-Identifier: Apache-2.0
```

See the included [`LICENSE`](LICENSE) file for full text.

---

_Updated: 2025-08-12_

