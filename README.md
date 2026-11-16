# Acme Operations Assistant
An AI agent application for **Acme** powered by a local **Phi-4** model via **Ollama**, featuring role-based skills (`admin`, `sales_user`, `support_user`), an SSE-based MCP server, Keycloak authentication, PostgreSQL storage, Redis persistence, and a Streamlit UI.

## Table of Contents

- [1. Install and Start Phi-4 (Ollama)](#1-install-and-start-phi-4-ollama)
- [2. Install and Start the Agent Stack](#2-install-and-start-the-agent-stack)
- [3. Keycloak Test Users & Roles](#3-keycloak-test-users--roles)
- [4. Accessing Services](#4-accessing-services)
- [5. Project Structure & Role Skills](#5-project-structure--role-skills)

---

## 1. Install and Start Phi-4 (Ollama)

The `app` service connects to Ollama at `http://host.docker.internal:11434` by default using `SLM_NAME=phi4`.

1. **Install Ollama**:
   - **macOS / Windows**: Download and install the Ollama desktop app from [ollama.com](https://ollama.com).
   - **Linux**: Install via the official script:
     ```bash
     curl -L [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
     ```
    
2. **Start the Ollama service** (if not already running in the background):
   ```bash
   ollama serve

3. **Pull and verify Phi-4**:
    ollama pull phi4
    ollama list
    ollama run phi4


## 2. Install and Start the Agent Stack

1. Clone/open the ACME repository and ensure Docker and Docker Compose are installed.
2. Configure .env in the repository root
3. Prepare Streamlit secrets: streamlit/.streamlit/secrets.toml
4. Build and start all services: 
    docker compose up -d --build
5. Check container health and logs:
    docker compose ps
    docker compose logs -f app
6. Stop services:
    docker compose down  


## 3. Keycloak Test Users & Roles
Keycloak runs in start-dev mode on port 8080 (http://localhost:8080) using the master realm (KEYCLOAK_ISSUER_URL=http://localhost:8080/realms/master).

**Bootstrap Admin Account**:
Sign in to the Keycloak Admin Console using the bootstrap credentials defined in your .env file:   
URL: http://localhost:8080   Username: ${KEYCLOAK_ADMIN}   Password: ${KEYCLOAK_ADMIN_PASSWORD}

**Role permissions**:
- sales_user: Read-only customer/issues
- support_user: Read & update issues
- admin: Full permissions

## 4. Accessing Services

Once `docker compose up -d --build` is running, services are available at:

**| Service | Container Name | URL / Endpoint | Notes |**
| :--- | :--- | :--- | :--- |

| **Streamlit UI** | `local_streamlit_service` | `http://localhost:8501` | Main user interface; forwards auth via `forward_keycloak.py` and calls `app` at `http://app:8000` |
| **Agent API (`app`)** | `local_app_service` | `http://localhost:8000` | Python agent backend (`app/main.py`) connected to Ollama, PostgreSQL, Redis, Keycloak, and MCP |
| **Keycloak** | `local_keycloak_service` | `http://localhost:8080` | Identity provider (`master` realm) backed by PostgreSQL (`keycloak` database) |
| **MCP Server** | `local_mcp_service` | `http://localhost:3001` (host) / `http://mcp-server:3000/sse` (internal) | Node.js MCP server (`mcp-server/index.js`) exposing `/sse` and querying PostgreSQL |
| **Ollama (Phi-4)** | Host / Local process | `http://localhost:11434` (`http://host.docker.internal:11434` from containers) | Serves local `phi4` model |
| **PostgreSQL** | `local_postgres_service` | `localhost:5432` | Initialized with `./postgresql/init.sql`; persisted in `postgres_data` volume |
| **Redis** | `local_redis_service` | `localhost:6379` | Runs with `--appendonly yes`; persisted in `redis_data` volume |

## 5. Project Structure & Role Skills
ACME/
├── app/                          # Python agent backend (main.py, dockerfile, requirements.txt)
├── keycloak/                     # Keycloak realm / configuration files
├── mcp-server/                   # Node.js MCP server (index.js, dockerfile, package.json)
├── postgresql/                   # PostgreSQL data & init.sql
├── redis/data/                   # Local Redis data files (appendonlydir, dump.rdb)
├── skills/                       # Mounted read-only at /app/skills:ro
│   ├── admin.md
│   ├── common.md
│   ├── sales_user.md
│   └── support_user.md
├── streamlit/                    # Streamlit frontend (app.py, forward_keycloak.py, .streamlit/secrets.toml)
├── .env                          # Environment variables (not committed)
└── docker-compose.yml            # Multi-service orchestration

