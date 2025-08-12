Maxey0 — Enterprise Agentic Memory & Modular AI Orchestration
Maxey0 is an agentic memory platform and modular AI orchestration framework for multi-agent systems, dynamic memory, and real-time ingestion. It stitches together Neo4j (graph memory & provenance), vector backends (Redis/FAISS), and tool-augmented agents behind clean FastAPI services. 
________________________________________
Core Philosophy
1.	Modularity first (swappable components)
2.	Stateful agents (persistent, queryable memory)
3.	Multi-modal context (text, structured, OCR)
4.	Continuous learning (real-time ingestion)
5.	Seamless orchestration (RAG, MCP, LangChain, CrewAI, custom adapters)
These principles underpin the repo layout and service boundaries. 
________________________________________
System Overview
•	Two-plane memory
o	Graph plane (Neo4j): provenance & topics (:Agent, :Memory, :Topic, [:WROTE], [:TAGGED])
o	Vector plane (Redis/FAISS): fast ANN per scope
•	Streaming context: Redis/Kafka; A2A messaging on Redis Streams (groups, ACK, DLQ, audit)
•	Optional GNN router: GraphSAGE/GIN (off the hot path)
•	Tool suite: web server, code interpreter sandbox, virtual env, web search, analytics dashboard, agent dev kit
•	Observability: dashboard across Redis/Neo4j; DLQ pressure & rates
•	Magnitude Spectrum mapping: deterministic, model-agnostic embeddings
•	No-placeholder policy: shipped modules are complete & tested. 
________________________________________
Repository Layout
text

Maxey0/
├── README.md
├── requirements.txt
├── pyproject.toml
├── LICENSE
├── .gitignore
├── docker/
│   ├── compose.full.yml
│   ├── Dockerfile.api
│   ├── Dockerfile.runner
│   ├── Dockerfile.gnn
│   ├── Dockerfile.web
│   └── init-neo4j.cypher
├── configs/
│   ├── settings.toml
│   ├── logging.yaml
│   └── secrets.env.example
├── data/
│   ├── vectors/
│   │   └── example_vectors.json
│   ├── conversations/
│   │   ├── maxey0_conversations.json
│   │   └── maxey00_conversations.json
│   ├── preembedded/
│   │   ├── maxey0_vectors.jsonl
│   │   └── maxey00_vectors.jsonl
│   └── agents/
│       ├── maxey_agents.cypher
│       └── agents_metadata.json
├── docs/
│   ├── architecture.md
│   ├── agent_lifecycle.md
│   ├── api_reference.md
│   └── developer_guide.md
├── scripts/
│   ├── ingest_knowledge.py
│   ├── ingest_conversations.py
│   ├── watch_ingest.py
│   └── no_placeholder_guard.py
├── Maxey0/
│   ├── __init__.py
│   ├── superagent.py
│   ├── core/
│   │   ├── config.py
│   │   ├── logging.py
│   │   └── utils.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── memory_api.py
│   │   ├── runner_api.py
│   │   └── mcp_api.py
│   ├── server/
│   │   ├── __init__.py
│   │   ├── endpoints_ops.py
│   │   ├── retriever.py
│   │   ├── runner.py
│   │   ├── a2a_streams.py
│   │   └── mcp_server.py
│   ├── a2a/
│   │   ├── __init__.py
│   │   ├── streams.py
│   │   ├── memory_sharing.py
│   │   └── cross_framework_adapter.py
│   ├── mms/
│   │   ├── __init__.py
│   │   └── memory_manager.py
│   ├── latent/
│   │   ├── __init__.py
│   │   ├── latent_space_map.py
│   │   └── magnitude_spectrum.py
│   ├── vectors/
│   │   ├── __init__.py
│   │   └── backends/
│   │       ├── __init__.py
│   │       ├── memory.py
│   │       ├── redis_scan.py
│   │       └── faiss_scan.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── web_server_tool.py
│   │   ├── code_interpreter_tool.py
│   │   ├── virtual_env_tool.py
│   │   ├── web_search_tool.py
│   │   ├── analytics_dashboard_tool.py
│   │   ├── ocr_tool.py
│   │   ├── pairwise_distance_tool.py
│   │   └── sandbox_tool.py
│   ├── adapters/
│   │   ├── __init__.py
│   │   ├── langchain_adapter.py
│   │   ├── crewai_adapter.py
│   │   ├── nvidia_agent_adapter.py
│   │   └── swarm_adapter.py
│   ├── ops/
│   │   ├── __init__.py
│   │   ├── drift_analytics.py
│   │   ├── observability.py
│   │   └── metrics_collector.py
│   └── neo4j/
│       ├── __init__.py
│       └── neo4j_client.py
└── tests/
    ├── __init__.py
    ├── test_memory_api.py
    ├── test_runner_api.py
    ├── test_a2a.py
    ├── test_latent_space_map.py
    ├── test_magnitude_spectrum.py
    ├── test_tools.py
    └── test_gnn_router.py
This is exactly the filesystem from readme1.txt (cleaned of “printed/not printed” flags). 
________________________________________
Prerequisites
•	Python 3.10+
•	Neo4j 5+ (local or Aura) with Bolt enabled
•	Redis 6+ (vector module or use FAISS backend)
•	Optional: Docker & Compose; PyTorch + PyG for the GNN router. 
________________________________________
Installation
Option A — uv (fast, reproducible)
bash
git clone https://github.com/TokenlessAI/Maxey0.git
cd Maxey0
uv pip install -r requirements.txt
Option B — venv + pip
bash

git clone https://github.com/TokenlessAI/Maxey0.git
cd Maxey0
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
________________________________________
Configuration
Create and edit environment files:
makefile

cp configs/secrets.env.example configs/secrets.env
# Minimal variables in configs/secrets.env
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=yourpassword
NEO4J_DATABASE=neo4j
REDIS_URL=redis://localhost:6379/0

# Optional
A2A_STREAM_NAMESPACE=m0
BRAVE_SEARCH_API_KEY=
Additional tuning lives in configs/settings.toml (embedding dims, mapper selection). 
Initialize Neo4j constraints/indexes (first run only) if you’re not using Compose bootstrapping:
bash

cypher-shell -a bolt://localhost:7687 -u neo4j -p yourpassword < docker/init-neo4j.cypher
________________________________________
Run with Docker Compose
Bring up Neo4j, Redis, and the APIs:
bash

docker compose -f docker/compose.full.yml up -d
Services (when started):
•	Memory API → http://localhost:18080
•	Runner API → http://localhost:18081
•	Analytics Dashboard Tool → http://localhost:8090
•	Web Server Tool → http://localhost:8080
Shut down:
bash

docker compose -f docker/compose.full.yml down -v
________________________________________
Start Services Manually
bash

# Memory API
uvicorn Maxey0.api.memory_api:app --host 0.0.0.0 --port 18080

# Runner API
uvicorn Maxey0.api.runner_api:app --host 0.0.0.0 --port 18081
________________________________________
Core APIs (selected)
Memory API (http://localhost:18080)
•	Insert vectors: POST /memory/insert
•	Insert text (auto-embed): POST /memory/insert_text
•	Query (KNN + provenance join): POST /memory/query
•	Patch metadata: POST /memory/meta
•	Delete: POST /memory/delete 
Runner API (http://localhost:18081)
•	Execute a tool module: POST /run (returns a structured {"result": ...}; supports multi-step plans + Neo4j audit) 
________________________________________
Tools (programmatic usage)
•	Web Server Tool: serve HITL workspaces/artifacts/logs
•	Code Interpreter Tool: sandboxed Python with optional ephemeral venv
•	Virtual Env Tool: create/install/info for agent venvs
•	Web Search Tool: DDG by default; Brave if key present
•	Analytics Dashboard Tool: live metrics via Redis/Neo4j
•	Agent Development Kit: define/register/spawn agents dynamically
Each tool exposes a single run(**kwargs) -> dict entrypoint. 
________________________________________
Latent Space Mapping
Default embeddings use the Magnitude Spectrum Generator (Maxey0/latent/magnitude_spectrum.py) → deterministic, robust, dimension-tunable vectors. Access via LatentSpaceMapper (latent_space_map.py):
embed(text) -> List[float] (default) and encode(text) -> List[float]. Switch mappers via config. 
________________________________________
Neo4j Graph Schema (provenance)
•	Unique constraints: (:Memory {id}), (:Agent {name}), (:Topic {name})
•	Core pattern:
(:Agent {…})-[:WROTE]->(:Memory {…})
(:Memory)-[:TAGGED]->(:Topic {name})
Provenance hydration is automatic when with_provenance=true on memory queries. 
________________________________________
A2A Messaging (Redis Streams)
•	Main stream: ${A2A_STREAM_NAMESPACE}:a2a (default m0:a2a)
•	DLQ: ${NS}:dlq
•	Audit: ${NS}:audit
Consumer groups, ACK, retries → DLQ on failure; dashboard computes backlog & rates. 
________________________________________
Testing
bash

pytest -q
# Optional guard to enforce “no placeholders”:
python scripts/no_placeholder_guard.py Maxey0
Covers import/config, memory ops (in-mem/Redis), tool entrypoints, latent determinism. 
________________________________________
Production Hardening (highlights)
•	Secrets via a manager; never commit .env
•	TLS termination at a proxy; RBAC + JWT/API keys for APIs
•	Neo4j: auth, least-privilege, backups, prefer managed Aura
•	Redis: ACLs + TLS; prefer managed (ElastiCache/MemoryDB/Redis Cloud)
•	Resource limits & ulimits; gate “dangerous” tools behind privileged scopes
•	Ship logs to ELK/CloudWatch; expose metrics (Prometheus) if extended. 
________________________________________
Troubleshooting (quick picks)
•	Memory API 500s: check Neo4j URI/user/pass/database; ensure constraints exist
•	Empty vector results: verify backend reachable; embedding dims match index config
•	Runner tool errors: check import path/params; inspect logs/audit
•	Dashboard says “neo4j disabled”: ensure Python Neo4j driver installed; env set
•	Web server port in use: stop existing or change port. 
________________________________________
License
Apache License, Version 2.0 (Apache-2.0).
This project is licensed under the Apache License, Version 2.0 

