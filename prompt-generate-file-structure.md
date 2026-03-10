# PROMPT: Generate Complete Project File Structure

## How to Use

1. Copy everything below (starting from `---START OF PROMPT---`)
2. Attach the file `tech-plan-ai-consultations-EN.docx` (your technical plan)
3. Paste into a new chat session with Claude or ChatGPT

---

## ---START OF PROMPT---

### Role

You are a Senior Fullstack Architect with 15+ years of experience designing high-load microservice systems. Your expertise covers Python (FastAPI), TypeScript (Next.js), DevOps (Docker/Kubernetes), and Clean Architecture.

You follow these strict rules:
- Use ONLY proven, battle-tested technologies from official documentation.
- Every library you mention MUST exist on PyPI or npm — never invent libraries.
- Reference best practices from real open-source projects: `tiangolo/full-stack-fastapi-template`, `vercel/next.js/examples`.
- Do NOT rely on forum opinions, unverified blog posts, or hearsay. Use only official docs, reputable technical books, and well-reviewed articles by established engineers.

---

### Context

I am attaching the full technical plan for an **AI Consultation Platform** with 50+ professional specializations (lawyer, psychologist, physician, financial advisor, etc.).

**Read the attached document completely before starting.**

The project consists of:

| Component | Technology | Description |
|-----------|-----------|-------------|
| 7 Microservices | Python 3.12+ / FastAPI | users, specialists, consultation, payment, rag, bot, api-gateway |
| Frontend | Next.js 14+ / TypeScript / Tailwind CSS | PWA-compatible web application |
| Telegram Bot | aiogram 3.x (Python) | Bot with FSM, inline keyboards, payments |
| RAG Pipeline | Qdrant + sentence-transformers + LLM | Vector knowledge base for 50+ specializations |
| Architecture | Clean Architecture | Entities → Use Cases → Adapters → Frameworks |

---

### Task

Generate the **complete file structure** for this project — every directory and every file across all components. Do not skip anything.

---

### Structural Requirements

#### 1. Each Microservice (×7)

Every microservice MUST follow this Clean Architecture template strictly:

```
service-name/
├── domain/
│   ├── __init__.py
│   ├── entities/
│   │   ├── __init__.py
│   │   └── [entity].py           # Rich Entity with encapsulated business logic
│   └── value_objects.py          # Value Objects (Email, Money, ConfidenceScore, etc.)
├── application/
│   ├── __init__.py
│   ├── use_cases/
│   │   ├── __init__.py
│   │   └── [use_case].py        # One file per use case (Interactor)
│   ├── dtos.py                   # Input/Output DTOs (no framework annotations)
│   └── interfaces.py             # ABC interfaces for repositories and gateways
├── adapters/
│   ├── __init__.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── routes.py             # FastAPI endpoint definitions
│   │   ├── schemas.py            # Pydantic validation schemas
│   │   ├── controllers.py        # Request → DTO mapping
│   │   ├── dependencies.py       # FastAPI Depends() injection
│   │   └── middleware.py         # Rate limiting, auth, request logging
│   └── db/
│       ├── __init__.py
│       ├── models.py             # SQLAlchemy ORM models (Storage Models)
│       ├── repos.py              # Repository implementations (DB ↔ Entity)
│       └── mappers.py            # ORM Model ↔ Domain Entity mapping
├── infra/
│   ├── __init__.py
│   ├── database.py               # Async engine, session factory, connection pool
│   ├── config.py                 # Pydantic Settings (reads from .env)
│   ├── redis_client.py           # Redis connection and helpers
│   └── logging_config.py         # Structured JSON logging (structlog)
├── tests/
│   ├── __init__.py
│   ├── unit/                     # Entity + Use Case tests (no infrastructure)
│   │   └── ...
│   ├── integration/              # Tests with real DB
│   │   └── ...
│   └── conftest.py               # Shared pytest fixtures
├── main.py                       # FastAPI application entry point
├── Dockerfile
├── requirements.txt              # Pinned versions (e.g., fastapi==0.111.0)
├── alembic.ini
├── alembic/
│   ├── env.py
│   └── versions/                 # Database migration files
├── .env.example
└── README.md
```

For each of the 7 microservices, list the **specific** entity files, use case files, and route files relevant to that service. Do not use generic placeholders — name every file.

The 7 microservices and their specific responsibilities:

| Service | Entities | Key Use Cases |
|---------|---------|---------------|
| users-service | User, Profile | register_user, authenticate, update_profile |
| specialists-service | Specialist, Category | list_specialists, get_specialist, manage_categories |
| consultation-service | Consultation, Message | create_consultation, send_message, get_history |
| payment-service | Payment, Subscription | process_payment, manage_subscription, check_limits |
| rag-service | Document, Embedding, Collection | rag_query, ingest_document, manage_collection |
| bot-service | (uses shared DTOs) | Telegram bot handlers |
| api-gateway | (no entities) | Route aggregation, auth middleware, rate limiting |

---

#### 2. Frontend (Next.js 14+ with App Router)

Required structure:

- **Pages (app/):** home, specialists catalog, chat/[id], payment, profile, admin dashboard, auth (login/register)
- **Components:** ChatWindow, SpecialistCard, PaymentForm, Header, Footer, Sidebar, MessageBubble, TypingIndicator, SearchBar
- **Hooks:** useAuth, useChat, usePayment, useWebSocket, useSpecialists
- **API client:** typed fetch wrapper (`lib/api.ts`), response types (`lib/types.ts`)
- **PWA:** manifest.json, service worker (sw.js)
- **Auth middleware:** Next.js middleware.ts for protected routes
- **Internationalization (i18n):** Russian + English minimum
- **State management:** Context or Zustand
- **Error handling:** error.tsx, not-found.tsx, loading.tsx per route group

---

#### 3. Telegram Bot (aiogram 3.x)

Required structure:

- **Handlers:** start, help, specialist selection, consultation dialogue, payment, profile
- **FSM states:** defined per dialogue flow
- **Keyboards:** inline keyboards for specialist selection, reply keyboards for main menu
- **HTTP client:** async client to communicate with backend API
- **Middleware:** throttling (anti-flood), logging, user identification
- **Config:** environment-based settings
- **Webhook:** production webhook setup (not polling)

---

#### 4. Admin Panel

Include a dedicated admin interface with:

- **Content management:** CRUD operations for specialists, categories, RAG documents, FAQ
- **Role-based access control (RBAC):**
  - Admin — full access (user management, system settings, logs)
  - Moderator — content moderation, user warnings/bans
  - Content Manager — specialist profiles, RAG document uploads only
- **Integration tools:**
  - Google Docs API import (for content ingestion)
  - Bulk upload interface for RAG vector DB documents
  - Grafana/Prometheus metrics dashboard embed
  - User management panel (ban, unban, change subscription tier)
- **Monitoring:** live logs viewer, health-check status, request statistics

---

#### 5. Security (mandatory files and components)

Every security measure below MUST be reflected in the file structure:

| Security Layer | Implementation | Files Required |
|----------------|---------------|----------------|
| Rate Limiting | slowapi or custom middleware | Each service: `middleware.py` |
| CAPTCHA | hCaptcha or reCAPTCHA v3 | Frontend: captcha component; Backend: verification endpoint |
| Input Validation | Pydantic schemas on every endpoint + HTML sanitization | `schemas.py` per service |
| CORS | Configured allowed origins | `main.py` CORS middleware |
| Security Headers | CSP, X-Frame-Options, HSTS, X-Content-Type-Options | Nginx config + middleware |
| DDoS Protection | Cloudflare + Nginx rate limiting | `nginx/nginx.conf` |
| Authentication | JWT + Refresh tokens, OAuth2 (Google, Telegram) | `auth/` module in users-service |
| Password Encryption | bcrypt | users-service entity logic |
| Data Encryption | AES for sensitive fields | `encryption.py` utility |
| CSRF Protection | Token-based CSRF for forms | Frontend middleware + backend validation |
| SQL Injection | ORM with parameterized queries (SQLAlchemy) | Already handled by ORM layer |
| Secrets Management | .env files + optional HashiCorp Vault | `.env.example` per service, `vault_client.py` |

---

#### 6. Infrastructure and DevOps

Required files:

- **docker-compose.yml** — full orchestration: all 7 services + PostgreSQL + Redis + Qdrant + Nginx (both dev and prod profiles)
- **docker-compose.override.yml** — development overrides (hot reload, debug ports)
- **k8s/** — Kubernetes manifests:
  - Deployment per service
  - Service (ClusterIP)
  - Ingress (with TLS)
  - HPA (Horizontal Pod Autoscaler) per service
  - ConfigMap and Secret per service
  - PersistentVolumeClaim for PostgreSQL and Qdrant
- **nginx/** — reverse proxy configuration:
  - `nginx.conf` — upstream load balancing, gzip, keepalive, SSL termination
  - `conf.d/` — per-service proxy configs
  - `ssl/` — certificate directory
- **.github/workflows/** — CI/CD pipelines:
  - `lint.yml` — ruff/flake8 + eslint
  - `test.yml` — pytest + jest
  - `build.yml` — Docker image builds
  - `deploy.yml` — deployment to staging/production
- **monitoring/**:
  - `prometheus.yml` — scrape configuration
  - `grafana/dashboards/` — pre-configured dashboard JSON files
  - `grafana/provisioning/` — datasource and dashboard provisioning
  - `alertmanager.yml` — alert rules (service down, high latency, error rate)
- **scripts/**:
  - `backup_postgres.sh` — automated PostgreSQL backup (pg_dump)
  - `backup_qdrant.sh` — Qdrant snapshot script
  - `restore_postgres.sh` — database restore
  - `restore_qdrant.sh` — vector DB restore
  - `seed_data.py` — populate DB with initial data (specialists, categories)
  - `migrate_all.sh` — run Alembic migrations for all services
- **Makefile** — commands: `make build`, `make up`, `make test`, `make lint`, `make migrate`, `make backup`, `make deploy`

---

#### 7. Logging and Backups

- **Structured JSON logging:** structlog library in every Python service
- **Centralized log collection:** ELK stack configuration files OR Grafana Loki config
- **PostgreSQL backups:** pg_dump cron scripts with retention policy
- **Vector DB backups:** Qdrant snapshot automation
- **Log rotation:** logrotate configuration

---

### Load Requirements (10,000 Concurrent Users)

The architecture must support 10,000 simultaneous users. Include the following in the structure:

| Requirement | Solution | Where It Appears |
|-------------|---------|-----------------|
| Connection pooling | asyncpg + SQLAlchemy async (pool_size=20, max_overflow=40) | `infra/database.py` |
| Caching | Redis for sessions, RAG results, rate limit counters | `infra/redis_client.py` |
| Async task queue | Celery + Redis broker (email, payment webhooks) | `workers/` directory per service |
| Load balancing | Nginx upstream config with keepalive | `nginx/nginx.conf` |
| Auto-scaling | Kubernetes HPA (CPU/memory-based) | `k8s/hpa.yml` |
| Real-time chat | FastAPI WebSocket | `adapters/api/websocket.py` in consultation-service |
| Static assets | CDN via Cloudflare or Vercel Edge | Frontend deployment config |

---

### Response Format

Complete this task in **3 stages.** Do not skip any stage.

---

**STAGE 1: Annotated File Tree**

Output the complete project file tree. After each file, add a brief comment (1 line max) explaining its purpose.

Format:
```
ai-consultation-platform/
├── services/
│   ├── users-service/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── __init__.py
│   │   │   │   └── user.py              # Rich Entity: has_free_requests(), can_access(), validate_email()
│   │   │   └── value_objects.py          # Email, HashedPassword, UserId
│   │   ├── application/
│   │   │   ├── use_cases/
│   │   │   │   ├── register_user.py      # User registration with email verification
│   │   │   │   └── authenticate.py       # JWT authentication flow
│   ... (continue for ALL files in ALL services, frontend, bot, infra)
```

**IMPORTANT:** List EVERY file. Do not use `...` or "similar structure" shortcuts. Expand all 7 microservices fully.

---

**STAGE 2: File Description Tables**

For each module (microservice, frontend, bot, infra), provide a table:

| File Path | Purpose | Clean Architecture Layer | Key Classes / Functions | Dependencies |
|-----------|---------|------------------------|------------------------|-------------|
| domain/entities/user.py | User entity with business rules | Entity | User, has_free_requests(), can_access() | None (pure Python) |
| application/use_cases/register_user.py | Registration orchestration | Use Case | RegisterUser.execute() | IUserRepository (ABC) |
| adapters/api/routes.py | HTTP endpoint definitions | Adapter | POST /register, POST /login | FastAPI, schemas.py |

---

**STAGE 3: Root README.md**

Generate a complete `README.md` for the project root with these sections:

1. **Project Overview** — what the platform does, key features
2. **Architecture** — Clean Architecture explanation + ASCII diagram showing layer dependencies
3. **Technology Stack** — table with all technologies and their purposes
4. **Quick Start** — step-by-step: clone → configure .env → `docker-compose up` → access at localhost
5. **Project Structure** — condensed file tree (top 2-3 levels)
6. **API Documentation** — links to auto-generated OpenAPI docs per service
7. **Development Guide** — how to run locally, hot reload, run individual services
8. **Testing** — how to run unit tests, integration tests, E2E tests; target coverage ≥ 80%
9. **Deployment** — Docker → Kubernetes → CI/CD pipeline description
10. **Monitoring** — Prometheus + Grafana setup instructions
11. **Contributing** — branch naming, commit conventions, PR process

---

### Constraints (STRICTLY ENFORCED)

1. **Official sources only.** Every technology choice must come from official documentation: [FastAPI](https://fastapi.tiangolo.com), [Next.js](https://nextjs.org/docs), [aiogram](https://docs.aiogram.dev), [SQLAlchemy](https://docs.sqlalchemy.org), [Qdrant](https://qdrant.tech/documentation), [Docker](https://docs.docker.com), [Kubernetes](https://kubernetes.io/docs).

2. **No fictional libraries.** Every package must exist on PyPI (`pip install`) or npm (`npm install`). If unsure, do not include it.

3. **Benchmark against real projects.** Cross-reference your structure with:
   - `tiangolo/full-stack-fastapi-template` (Python backend)
   - `vercel/next.js/examples` (Next.js frontend)
   - `pydantic/pydantic` (project structure)

4. **No missing files.** It is better to include an extra file than to forget a necessary one. After generating the tree, re-check each microservice against the template.

5. **Pin library versions.** Specify exact stable versions in `requirements.txt` and `package.json`. Use the latest stable release as of March 2026.

6. **Separate `__init__.py` files.** Every Python package directory must include `__init__.py`.

---

### Attached File

Technical plan: `tech-plan-ai-consultations-EN.docx`

Read it completely before starting. All business logic, technology stack, architecture decisions, and sprint breakdown are defined there.

## ---END OF PROMPT---
