# PROJECT TECHNICAL PLAN

**Microservice Web Application Based on Clean Architecture**

AI Consultation Platform with 50+ Professional Specializations

RAG + Vector DB | Telegram Bot | PWA | Cross-platform

Version 1.0

---

## Section 1. Application Overview

The platform provides clients with access to ~50 professional specializations (lawyer, psychologist, financial advisor, physician, etc.), each represented by an AI agent with RAG (vector knowledge base).

### Key Business Rules

- **Free trial:** the first 10 requests to any specialist are free. After that, payment is required.
- **AI agents with RAG:** each specialist is an LLM agent enriched with a vector database of professional knowledge.
- **Confidence threshold:** if the confidence score < 0.87, the request is escalated to a real expert.
- **Multiplatform:** Web (PWA), Android, iOS, Windows, macOS, Linux, Telegram Bot.

> **What:** Define the core business logic of the consultation platform.
>
> **Why:** Lock in key invariants (request limits, AI threshold, monetization) at the planning stage.
>
> **How:** Formalize business rules as Rich Entities (Clean Architecture).
>
> **Result:** Documented business rules in the Entities layer.
>
> **Verification:** Rules are verified with unit tests without infrastructure.

---

## Section 2. Project Architecture (Clean Architecture)

The project is built on 4 key layers of Clean Architecture. Dependencies are directed strictly inward: Frameworks → Adapters → Use Cases → Entities.

### 2.1. Entities Layer

The central layer — Enterprise Business Rules. Contains Rich Domain Objects with encapsulated business logic. Independent of any frameworks.

| File | Language | Purpose |
|------|----------|---------|
| `user.py` | Python | User entity (`has_free_requests()`, `can_access()`) |
| `specialist.py` | Python | Specialist entity (AI agent, profession, consultation methods) |
| `consultation.py` | Python | Consultation entity (messages, `confidence_score`, status) |
| `payment.py` | Python | Payment entity (`process()`, `validate_amount()`) |
| `subscription.py` | Python | Subscription entity (`is_active()`, `renew()`, `cancel()`) |

> **What:** Create Rich Entities for each domain object.
>
> **Why:** Encapsulate business rules inside entities (not in services), ensuring framework independence.
>
> **How:** Python dataclass/Pydantic without ORM annotations. Validation methods inside the class.
>
> **Result:** Classes with encapsulated logic, testable without a database.
>
> **Verification:** Unit tests for each Entity method without infrastructure mocks.

### 2.2. Use Cases Layer (Interactors)

The Application Specific Business Rules layer. Interactors orchestrate entities and repositories but contain no business rules. Input/output via DTOs (Boundaries).

| File | Language | Purpose |
|------|----------|---------|
| `create_consultation.py` | Python | Create consultation with AI agent |
| `process_payment.py` | Python | Process payment and verify limits |
| `escalate_to_expert.py` | Python | Escalation when confidence < 0.87 |
| `register_user.py` | Python | Registration and verification |
| `rag_query.py` | Python | RAG query to vector DB + LLM |
| `manage_subscription.py` | Python | Manage user subscription |
| `dtos.py` | Python | Clean DTOs without framework annotations |
| `interfaces.py` | Python | ABC interfaces for repositories and gateways |

> **What:** Implement interactors for each use case scenario.
>
> **Why:** Use Cases must not contain business logic — only orchestration. This enables predictable AI code generation.
>
> **How:** Each Use Case: retrieve data via interface → coordinate Entity → save via repository.
>
> **Result:** Isolated scenarios, testable with mock repositories.
>
> **Verification:** Each Use Case covered by tests with mock IRepository. Verify: is `save()` called, is the result correct.

### 2.3. Interface Adapters Layer

The bridging layer between the external world and the core. Controllers, repositories, presenters. Data mapping between formats occurs here.

| File | Language | Purpose |
|------|----------|---------|
| `routes.py` | Python | FastAPI routes (HTTP → Use Case) |
| `controllers.py` | Python | Controllers (mapping Request → DTO) |
| `schemas.py` | Python | Pydantic validation schemas for API |
| `sqlalchemy_repos.py` | Python | Repository implementations (DB ↔ Entity) |
| `presenters.py` | Python | Transform Entity → Response DTO |
| `mappers.py` | Python | Mapping ORM Model ↔ Domain Entity |

### 2.4. Frameworks & Drivers Layer

The outermost and most volatile layer: frameworks, databases, UI, external services. Can be replaced without modifying the core.

| File | Language | Purpose |
|------|----------|---------|
| `main.py` | Python | FastAPI service entry point |
| `database.py` | Python | SQLAlchemy engine, session, migrations (Alembic) |
| `models.py` | Python | ORM models (Storage Models) for PostgreSQL |
| `bot.py` | Python | Telegram bot (aiogram/telebot) |
| `rag_engine.py` | Python | Integration with vector DB (Qdrant/Pinecone) |
| `llm_client.py` | Python | LLM API client (OpenAI/Anthropic) |
| `payment_gateway.py` | Python | Payment system adapter (Stripe/YooKassa) |
| `config.py` / `.env` | Python/ENV | Environment configuration |
| `Dockerfile` | Docker | Service containerization |
| `docker-compose.yml` | YAML | Microservice orchestration |

---

## Section 3. Technology Stack

### 3.1. Backend

| Technology | Purpose |
|-----------|---------|
| Python 3.12+ / PyPy | Primary backend language |
| FastAPI | Asynchronous web framework (async/await, auto-generated OpenAPI docs) |
| PostgreSQL | Primary database (users, consultations, payments) |
| SQLAlchemy + Alembic | ORM and database migrations |
| Qdrant / Pinecone | Vector database for the RAG pipeline |
| Redis | Caching, sessions, task queues (Celery) |
| aiogram 3.x | Telegram bot framework (async) |

### 3.2. Frontend

| Technology | Purpose |
|-----------|---------|
| Next.js 14+ | React framework (SSR, SSG, API Routes) |
| TypeScript | Strict frontend typing |
| Tailwind CSS | Utility-first CSS framework |
| next-pwa | Progressive Web App (mobile adaptation) |

### 3.3. Infrastructure & DevOps

| Technology | Purpose |
|-----------|---------|
| Docker + Compose | Microservice containerization |
| Kubernetes | Production orchestration |
| GitHub Actions / GitLab CI | CI/CD pipeline |
| Nginx | Reverse proxy, SSL, load balancing |

> **What:** Define the technology stack for all application layers.
>
> **Why:** Frameworks are implementation details (Clean Arch). The choice must account for replaceability.
>
> **How:** FastAPI for async and OpenAPI; Next.js for SSR/PWA; PostgreSQL as a reliable RDBMS; Qdrant for vector search.
>
> **Result:** A complete stack supporting asynchronicity and scalability.
>
> **Verification:** Each framework is used only in the Frameworks layer, never penetrating into Entity/Use Cases.

---

## Section 4. Complete Project File List

### 4.1. Backend Microservice Structure (example: consultation-service)

Each microservice replicates this structure. Microservices: **users-service, specialists-service, consultation-service, payment-service, rag-service, bot-service, gateway.**

| Path / File | Language | Layer | Purpose |
|------------|----------|-------|---------|
| `domain/entities/` | Python | Entity | Business entities |
| `domain/value_objects.py` | Python | Entity | Value objects |
| `application/use_cases/` | Python | Use Case | Interactors |
| `application/dtos.py` | Python | Use Case | Input/output DTOs |
| `application/interfaces.py` | Python | Use Case | ABC interfaces |
| `adapters/api/routes.py` | Python | Adapter | HTTP endpoints |
| `adapters/api/schemas.py` | Python | Adapter | Pydantic schemas |
| `adapters/db/models.py` | Python | Adapter | ORM models |
| `adapters/db/repos.py` | Python | Adapter | DB repositories |
| `adapters/db/mappers.py` | Python | Adapter | ORM↔Entity mapping |
| `infra/database.py` | Python | Framework | DB connection |
| `infra/config.py` | Python | Framework | Configuration |
| `main.py` | Python | Framework | Entry point |
| `Dockerfile` | Docker | Framework | Container |
| `requirements.txt` | pip | Framework | Dependencies |
| `tests/` | Python | All | Unit + Integration |

### 4.2. Frontend Structure (Next.js)

| Path / File | Language | Purpose |
|------------|----------|---------|
| `package.json` | JSON | Project dependencies |
| `next.config.js` | JavaScript | Next.js + PWA configuration |
| `tsconfig.json` | JSON | TypeScript settings |
| `tailwind.config.ts` | TypeScript | Tailwind CSS configuration |
| `app/layout.tsx` | TypeScript | Root application layout |
| `app/page.tsx` | TypeScript | Home page |
| `app/specialists/page.tsx` | TypeScript | Specialist selection page |
| `app/chat/[id]/page.tsx` | TypeScript | Chat with AI consultant |
| `app/payment/page.tsx` | TypeScript | Payment page |
| `components/ChatWindow.tsx` | TypeScript | Chat component |
| `components/SpecialistCard.tsx` | TypeScript | Specialist card |
| `components/PaymentForm.tsx` | TypeScript | Payment form |
| `lib/api.ts` | TypeScript | API client (fetch) |
| `lib/types.ts` | TypeScript | Data types |
| `public/manifest.json` | JSON | PWA manifest |
| `public/sw.js` | JavaScript | Service Worker |
| `.env.local` | ENV | Environment variables |

### 4.3. Telegram Bot Structure

| File | Language | Purpose |
|------|----------|---------|
| `bot.py` | Python | Bot entry point (aiogram) |
| `handlers/start.py` | Python | `/start`, `/help` handler |
| `handlers/consultation.py` | Python | AI specialist dialogue |
| `handlers/payment.py` | Python | Payment via bot |
| `keyboards.py` | Python | Inline keyboards |
| `states.py` | Python | FSM dialogue states |
| `api_client.py` | Python | HTTP client to backend |

### 4.4. Root Configuration Files

| File | Format | Purpose |
|------|--------|---------|
| `docker-compose.yml` | YAML | All services orchestration |
| `.env` | ENV | Global environment variables |
| `.gitignore` | Text | VCS exclusions |
| `Makefile` | Make | Build/run commands |
| `README.md` | Markdown | Project documentation |
| `k8s/` | YAML | Kubernetes manifests |
| `.github/workflows/` | YAML | CI/CD pipelines |

---

## Section 5. Development Stages (Sprints)

Breaking the project into phases allows staying within the LLM token limit during code generation: each phase produces a concrete result that can be verified before moving to the next. This is critical when working with ChatGPT/Claude: one sprint = one session/context.

### Sprint 1. Design & Prototyping (2 weeks)

> **What:** Design the microservice architecture and domain model.
>
> **Why:** Define service boundaries, API contracts, and domain entities before implementation begins.
>
> **How:** Identify 7 microservices: users, specialists, consultation, payment, rag, bot, gateway. Describe Rich Entities for each. Compose OpenAPI specifications.
>
> **Result:** Document: architecture diagram, entity list, API contracts, ERD.
>
> **Verification:** Peer-review of architecture. Verify: each service has clear boundaries and does not overlap with others.

### Sprint 2. Backend Service Implementation (4 weeks)

> **What:** Implement the core of each microservice: Entities → Use Cases → Adapters.
>
> **Why:** Layer-by-layer implementation ensures Dependency Rule compliance and enables AI to generate code layer by layer.
>
> **How:** 1) Write Rich Entities with tests. 2) Use Cases with mock repositories. 3) Adapters (FastAPI routes + SQLAlchemy repos). 4) Integration tests.
>
> **Result:** Working microservices with REST APIs, test coverage ≥ 80%.
>
> **Verification:** `pytest --cov` ≥ 80%. Verify: Entity imports nothing from external layers.

### Sprint 3. Frontend Development (3 weeks)

> **What:** Create a web application using Next.js + TypeScript with responsive design.
>
> **Why:** Provide a user interface for specialist selection, chat, and payment.
>
> **How:** Pages: home, specialist catalog, AI chat, payment, profile. Components: ChatWindow, SpecialistCard, PaymentForm.
>
> **Result:** PWA-compatible web application working on mobile and desktop.
>
> **Verification:** Lighthouse PWA audit ≥ 90. Offline functionality (Service Worker). Correct mobile rendering.

### Sprint 4. Multiplatform & Telegram Bot (3 weeks)

> **What:** Implement the Telegram bot, configure PWA, and create wrappers for desktop/mobile.
>
> **Why:** Ensure access across all platforms: Web, Android, iOS, Windows, macOS, Linux, Telegram.
>
> **How:** Bot on aiogram 3.x with FSM. PWA (next-pwa). Desktop: Tauri/Electron. Mobile: Capacitor/TWA.
>
> **Result:** Telegram bot with full functionality. PWA installable. Desktop builds available.
>
> **Verification:** Bot responds to `/start`, specialist selection, chat, payment. PWA Lighthouse ≥ 90.

### Sprint 5. AI & RAG Integration (3 weeks)

> **What:** Set up the RAG pipeline for 50 professional specializations.
>
> **Why:** AI agents must respond with expertise. With low confidence, escalation to a real expert occurs.
>
> **How:** 1) Load professional documents into vector DB (Qdrant). 2) Embedding pipeline (OpenAI/sentence-transformers). 3) RAG query: retrieval + LLM generation. 4) Confidence scoring. 5) Escalation when score < 0.87.
>
> **Result:** 50 AI agents, each with its own vector knowledge base. Escalation logic operational.
>
> **Verification:** A/B testing of response quality. Verification of escalation threshold on real queries.

### Sprint 6. Testing & Deployment (2 weeks)

> **What:** Comprehensive testing, CI/CD, and production deployment.
>
> **Why:** Ensure reliability, fault tolerance, and automated deployment.
>
> **How:** 1) E2E tests (Playwright). 2) Load testing (Locust). 3) CI/CD: GitHub Actions → Docker → K8s. 4) Monitoring (Prometheus + Grafana).
>
> **Result:** System in production with auto-deployment and monitoring.
>
> **Verification:** All services run in K8s. CI/CD pipeline passes without errors. Grafana alerts configured.

---

## Section 6. Multiplatform Support

| Platform | Technology | Description |
|----------|-----------|-------------|
| Web | Next.js + PWA | Responsive website with offline support |
| Android | TWA / Capacitor | PWA wrapper for Google Play |
| iOS | Capacitor / Safari PWA | PWA wrapper for App Store |
| Windows | Tauri / Electron | Native windowed application |
| macOS | Tauri / Electron | Native windowed application |
| Linux | Tauri / Electron | AppImage / Snap / Flatpak |
| Telegram | aiogram 3.x (Python) | Bot with inline keyboards, FSM, payments |

### 6.1. Telegram Bot: Key Features

The bot is implemented in Python using aiogram 3.x and communicates with the same backend via HTTP API. Key features:

- Asynchronous (async/await), high performance
- FSM (finite state machine) for multi-step dialogues
- Inline keyboards for specialist selection
- Payment integration via Telegram Payments API
- Webhooks instead of polling in production

> **What:** Ensure access on all target platforms.
>
> **Why:** Maximize user reach with minimal code duplication.
>
> **How:** PWA as the base platform → wrappers for native platforms. Telegram bot as a separate client.
>
> **Result:** 7 platforms with a unified backend. One UI codebase.
>
> **Verification:** Application launches on each target platform. Bot integration-tested.

---

## Section 7. AI Agents & RAG Pipeline

Each of the ~50 specialists is represented by an AI agent with an individual vector knowledge base.

### 7.1. Request Processing Pipeline

1. User sends a question via chat / Telegram
2. Question is converted to an embedding (sentence-transformers / OpenAI)
3. Search in vector DB (Qdrant) by the specialist's collection
4. Retrieved fragments + question → prompt for LLM
5. LLM generates response + confidence score
6. **If confidence ≥ 0.87** → response delivered to user
7. **If confidence < 0.87** → escalation to a real expert

> **What:** Implement the RAG pipeline with an escalation mechanism.
>
> **Why:** Ensure response quality and safe escalation when confidence is insufficient.
>
> **How:** Qdrant for vector DB, sentence-transformers for embeddings, LangChain/LlamaIndex for RAG orchestration.
>
> **Result:** RAG service with a configurable escalation threshold of 0.87.
>
> **Verification:** Testing: 100+ queries per specialist. Verify recall and precision.

---

## Section 8. LLM Token Limit Management

When using LLMs for code generation, it is critical not to overload the context window. Strategy:

- **One sprint = one session.** Each phase produces a self-contained result that can be verified.
- **Layer-by-layer generation.** First Entities, then Use Cases, then Adapters. Each layer fits within the token limit.
- **Explicit interfaces.** ABC interfaces are passed to the next session as a contract.
- **Structured prompts.** The What/Why/How/Result/Verification format minimizes ambiguity and repeated requests.

---

## Summary

This technical plan covers 7 microservices, 50+ AI agents with RAG, 7 target platforms, and 6 development sprints. The layered architecture (Clean Architecture) guarantees long-term manageability, testability, and framework independence. Each stage is structured according to the What → Why → How → Result → Verification principle, ensuring transparency and predictability of development.
