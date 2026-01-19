# Agent Service Toolkit

A clean, production-ready template for building and running **LangGraph** agents with a **FastAPI** service and a **Streamlit** UI. It shows the full path from agent definition → REST API (sync + streaming) → client → web app, with persistence, basic safety, and Dockerized dev workflow.

## What you get

- **LangGraph agents** using modern patterns (interrupt/resume, Command routing, Store-based memory)
- **FastAPI service** with `/invoke`, `/stream`, `/info`, `/history`, and `/feedback`
- **Streaming** over Server-Sent Events (SSE): token streaming, node updates, and custom events
- **Streamlit app**: chat UI with model/agent selection and live tool-call updates
- **Persistence**: thread-scoped conversation history + optional long-term memory (PostgreSQL)
- **RAG option**: simple Chroma DB pipeline for local document QA
- **Content moderation**: LlamaGuard wiring (optional)
- **Docker Compose** for dev and deployment, with hot-reload (`compose watch`)
- **Tests** and a lightweight client for quick scripts or integrations

## Architecture (at a glance)

**Flow:** Streamlit UI → lightweight client → FastAPI service → LangGraph agent(s) → tools/providers  
**Persistence:** Checkpointer for short-term conversation state, Store for long-term memory

| Layer | Key files | Responsibility |
| --- | --- | --- |
| UI | `src/streamlit_app.py` | Chat interface, streaming display, tool-call status, feedback |
| Client | `src/client/client.py` | Sync/async invoke, streaming parser, history/feedback helpers |
| Service | `src/service/service.py` | REST endpoints, SSE, lifespan init, state handling |
| Agents | `src/agents/*` | Agents (chat, research, RAG, supervisor, command, interrupt, etc.) |
| Core | `src/core/*`, `src/schema/*` | Settings, provider wiring, Pydantic models, enums |
| Memory | `src/memory/*` | Saver/Store implementations (SQLite/Postgres) |

## Quickstart (local, no Docker)

> Requires Python 3.11+

```bash
# Clone your repo
git clone https://github.com/Theepankumargandhi/agent-service-toolkit.git
cd agent-service-toolkit

# Create .env and add at least one LLM key
cp .env.example .env
# edit .env -> OPENAI_API_KEY=...

curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync --frozen
source .venv/bin/activate

# Run the FastAPI service
python src/run_service.py

# In a separate terminal, run the Streamlit app
source .venv/bin/activate
streamlit run src/streamlit_app.py

# Open the UI at http://localhost:8501
```

**Services**

- Streamlit UI: `http://localhost:8501`  
- Agent API: `http://0.0.0.0:8080` (OpenAPI docs at `/redoc`)  
- PostgreSQL: `localhost:5432`


```
# Running the agents
Streamlit UI

Choose an agent (chatbot, research-assistant, rag-assistant, supervisor, interrupt, etc.)

Select a model

Chat and watch live streaming as tokens and tool calls appear

Rate responses (posted to feedback endpoint if configured)

# Client

```
from client import AgentClient

client = AgentClient(agent="chatbot", base_url="http://0.0.0.0:8080")
response = client.invoke("Give me a fun fact about LangGraph.")
print(response.content)

```
# RAG
Place documents in ./data
Build a local Chroma DB:
```
python scripts/create_chroma_db.py
```
# Project structure
```
```
agent-service-toolkit/
├─ compose.yaml
├─ docker/
│  ├─ Dockerfile.service
│  └─ Dockerfile.app
├─ scripts/
│  └─ create_chroma_db.py
├─ src/
│  ├─ run_service.py
│  ├─ streamlit_app.py
│  ├─ client/client.py
│  ├─ service/service.py
│  ├─ agents/
│  │  ├─ agents.py
│  │  ├─ chatbot.py
│  │  ├─ research_assistant.py
│  │  ├─ rag_assistant.py
│  │  ├─ command_agent.py
│  │  ├─ interrupt_agent.py
│  │  ├─ langgraph_supervisor_agent.py
│  │  ├─ langgraph_supervisor_hierarchy_agent.py
│  │  └─ github_mcp_agent/
│  ├─ core/
│  ├─ memory/
│  └─ schema/
├─ tests/
├─ .env.example
└─ README.md
```

---

## Local Setup (Python)

> Requires Python 3.11+

```bash
# Clone the repository
git clone https://github.com/Theepankumargandhi/agent-service-toolkit.git
cd agent-service-toolkit

# Create and activate a virtual environment
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
# source .venv/bin/activate

# Install dependencies using uv or pip
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync --frozen
source .venv/bin/activate
```
# To run this 
```
python src/run_service.py
streamlit run src/streamlit_app.py
```
# Env configuration
```
OPENAI_API_KEY=your_openai_key
GROQ_API_KEY=optional_groq_key
POSTGRES_URI=postgresql://user:pass@host:5432/db
LANGSMITH_API_KEY=optional_for_tracing
AUTH_SECRET=optional_api_auth_key
```
The core design comes from JoshuaC215’s open-source work. 
