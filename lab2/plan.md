# Implementation Plan: E-Commerce Dashboard

**Branch**: `001-ecommerce-dashboard` | **Date**: 2025-10-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-ecommerce-dashboard/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Build a web dashboard for an e-commerce platform that displays data from an API gateway. The dashboard provides administrators with real-time visibility into three core business areas: Products (grid view showing inventory and prices), Users (list view showing customer accounts), and Orders (table view showing transaction statuses). This is a read-only dashboard focused on data visualization and monitoring, not data manipulation.

**Architecture**: Microservices with different programming languages per service to enable parallel development by multiple engineers. Each service (product-service, user-service, order-service) is independently deployable and developed by separate teams.

## Technical Context

### Multi-Service Architecture

**Frontend (Dashboard UI)**:
- **Language**: TypeScript 5.x (React 18.2+)
- **Framework**: PatternFly 5 (React components)
- **Build Tool**: Vite 5.x
- **Port**: 3001
- **Team**: Frontend Agent

**Product Service**:
- **Language**: Python 3.11+
- **Framework**: FastAPI 0.104+
- **Data**: Hardcoded mock data (10-20 products)
- **Port**: 8001
- **Team**: Product Agent

**User Service**:
- **Language**: Go 1.21+
- **Framework**: Gin 1.9+
- **Data**: Hardcoded mock data (10-20 users)
- **Port**: 8002
- **Team**: User Agent

**Order Service**:
- **Language**: Rust 1.75+
- **Framework**: Actix-web 4.x
- **Data**: Hardcoded mock data (10-20 orders)
- **Port**: 8003
- **Team**: Order Agent

**API Gateway**:
- **Language**: Node.js 20 LTS (TypeScript)
- **Framework**: Express 4.x
- **Port**: 8000
- **Team**: Gateway Agent
- **Purpose**: Route requests, aggregate responses, handle CORS, authentication

**Shared Infrastructure**:
- **Data**: Mock data hardcoded in each service (no external dependencies)
- **Containerization**: Podman 4.x+ / podman-compose
- **Testing**: Basic smoke tests per service
- **Target Platform**: Containerized deployment, modern web browsers

**Demo Goals**: Show multi-agent collaboration (parallel + sequential), polyglot microservices, Git integration workflow
**Constraints**: CORS-enabled at gateway, frontend port 3001+
**Scope**: Demo dashboard with 10-20 items per section, 7 independent agents working sequentially

## Agent Workflow & Code Integration

### Sequential Agent Execution

**Phase 1 - Backend Services** (⚡ RUN IN PARALLEL):
1. Product Agent → Commits to `product-agent` branch → **Merges to `master`**
2. User Agent → Commits to `user-agent` branch → **Merges to `master`**
3. Order Agent → Commits to `order-agent` branch → **Merges to `master`**

**All 3 backend agents work simultaneously** - no dependencies between them

**Phase 2 - Integration Layer**:
4. Gateway Agent → Commits to `gateway-agent` branch → **Merges to `master`**
5. Frontend Agent → Commits to `frontend-agent` branch → **Merges to `master`**

**Phase 3 - Deployment** (waits for ALL above):
6. DevOps Agent → **Works from `master` branch** → Verifies all 5 services exist → Creates compose.yaml → Commits to `devops-agent` branch → **Merges to `master`**

**Phase 4 - Quality Assurance** (waits for DevOps):
7. QA Agent → **Works from `master` branch** → Tests deployed system → Commits results to `qa-agent` branch → **Merges to `master`**

### Critical Integration Requirements

**Each Service Agent MUST**:
1. ✅ Complete all assigned tasks in their feature branch
2. ✅ Commit all changes with proper commit message
3. ✅ **MERGE their branch to `master`** before marking work complete
4. ✅ Verify merge successful: `git log master --oneline -5`

**DevOps Agent CANNOT start until**:
- All 5 Containerfiles exist in master branch
- All 5 service directories (product-service, user-service, order-service, api-gateway, frontend) exist in master branch
- All previous agents have merged their work

**QA Agent CANNOT start until**:
- DevOps Agent has merged compose.yaml to master branch
- All services are deployed and running via podman-compose

### Repository Structure Verification

Before DevOps Agent begins, the repository structure MUST look like:
```
/path/to/ecom/  (master branch)
├── product-service/
│   ├── Containerfile
│   ├── app/main.py
│   └── requirements.txt
├── user-service/
│   ├── Containerfile
│   ├── cmd/server/main.go
│   └── go.mod
├── order-service/
│   ├── Containerfile
│   ├── src/main.rs
│   └── Cargo.toml
├── api-gateway/
│   ├── Containerfile
│   ├── src/index.ts
│   └── package.json
├── frontend/
│   ├── Containerfile
│   ├── src/
│   └── package.json
└── specs/
    ├── tasks.md
    └── plan.md
```

**Missing Directories = Blocker**: If any service directory is missing, DevOps Agent MUST WAIT.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. API-First Architecture
**Status**: ✅ PASS
**Compliance**: Dashboard consumes data from existing API gateway. All data access goes through API endpoints. No direct database access from frontend.
**Action**: Ensure API contracts are documented in Phase 1.

### II. Data Integrity
**Status**: ✅ PASS (N/A)
**Compliance**: Mock data hardcoded in each service. No database required for demo.
**Action**: None required.

### III. Security by Default
**Status**: ✅ PASS (N/A)
**Compliance**: Authentication/authorization out of scope for demo.
**Action**: None required.

### IV. Test Coverage
**Status**: ✅ PASS (Simplified)
**Compliance**: Basic component tests, manual smoke testing.
**Action**: Each agent writes 2-3 simple tests for their service.

### V. Performance & Scale
**Status**: ✅ PASS (N/A)
**Compliance**: Demo with mock data (10-20 items per section). No performance requirements.
**Action**: None required.

### VI. Container-Based Deployment
**Status**: ✅ PASS
**Compliance**: Deployment via Podman/podman-compose. Frontend port 3001+ (not 3000).
**Action**: Create Containerfile for frontend. Create compose.yaml with resource limits. Frontend must run on port 3001 or higher.

### Technical Standards Compliance

- **CORS**: ✅ API gateway must enable CORS with explicit origin whitelisting
- **Port Allocation**: ✅ Frontend will use port 3001 (not 3000)
- **Node.js Dependencies**: ✅ All builds use `npm install` (never `npm ci`)
- **Containerization**: ✅ Podman/podman-compose exclusive, compose.yaml format

### Gate Decision - Post-Design Re-evaluation

**✅ FINAL PASS** - All constitution requirements addressed for multi-service architecture:

**Technical Decisions** (Demo Focus):
1. ✅ Multi-language services: Python (FastAPI), Go (Gin), Rust (Actix), TypeScript (React + Express)
2. ✅ Test frameworks: Minimal testing - basic smoke tests per service
3. ✅ Data: Hardcoded mock data (10-20 items per service)
4. ✅ Sequential workflow: Product → User → Order → Gateway → Frontend → DevOps → QA

**Constitution Compliance**:
5. ✅ **API-First**: Simple OpenAPI contracts define service interfaces
6. ✅ **Data Integrity**: N/A - mock data in each service
7. ✅ **Security**: CORS at gateway (explicit origin whitelist), auth out of scope for demo
8. ✅ **Test Coverage**: Basic smoke tests only
9. ✅ **Performance**: N/A - demo with small datasets
10. ✅ **Containerization**: Each service has Containerfile, compose.yaml orchestrates all services, Podman-only
11. ✅ **Port Allocation**: Frontend port 3001, backend services 8001-8003, gateway 8000
12. ✅ **Node.js Dependencies**: `npm install` enforced (never `npm ci`) for frontend and gateway

**Sequential Development Flow**:
- Agents work in sequence with clear handoff points
- Each service in separate directory with own Containerfile
- No shared code between services (only contract agreement)
- Each agent completes their work before next agent starts
- Total estimated duration: ~10-15 days for demo implementation

**Proceed to Phase 2: Task Generation (`/speckit.tasks`)**

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
# Microservices architecture - each service in separate directory
# Each agent works independently in their service directory

frontend/                          # Frontend Agent (TypeScript/React + PatternFly)
├── src/
│   ├── components/
│   │   ├── products/
│   │   │   ├── ProductGrid.tsx    # Uses PatternFly Gallery
│   │   │   └── ProductCard.tsx    # Uses PatternFly Card
│   │   ├── users/
│   │   │   ├── UserList.tsx       # Uses PatternFly List
│   │   │   └── UserListItem.tsx   # Uses PatternFly ListItem
│   │   ├── orders/
│   │   │   ├── OrderTable.tsx     # Uses PatternFly Table
│   │   │   ├── OrderRow.tsx
│   │   │   └── StatusBadge.tsx    # Uses PatternFly Label
│   │   ├── common/
│   │   │   ├── LoadingSpinner.tsx # Uses PatternFly Spinner
│   │   │   ├── ErrorMessage.tsx   # Uses PatternFly Alert
│   │   │   └── EmptyState.tsx     # Uses PatternFly EmptyState
│   │   └── layout/
│   │       └── Dashboard.tsx      # Uses PatternFly Page
│   ├── services/
│   │   ├── api.ts
│   │   ├── products.ts
│   │   ├── users.ts
│   │   └── orders.ts
│   ├── hooks/
│   │   └── useApiData.ts
│   ├── utils/
│   │   ├── formatters.ts
│   │   └── pagination.ts
│   ├── types/
│   │   └── api.types.ts     # Shared TypeScript interfaces
│   ├── App.tsx
│   └── main.tsx
├── tests/
│   ├── integration/
│   ├── components/
│   └── e2e/
├── Containerfile
├── package.json
├── tsconfig.json
└── vite.config.ts

product-service/                   # Product Agent (Python/FastAPI)
├── app/
│   ├── main.py              # FastAPI app entry point
│   ├── mock_data.py         # Hardcoded product data (10-20 items)
│   ├── schemas.py           # Pydantic schemas
│   └── routes.py            # API endpoints (GET /products)
├── tests/
│   └── test_products.py     # Basic smoke tests
├── Containerfile
└── requirements.txt

user-service/                      # User Agent (Go/Gin)
├── cmd/
│   └── server/
│       └── main.go          # Entry point
├── internal/
│   ├── models/
│   │   └── user.go
│   ├── handlers/
│   │   └── user_handler.go
│   └── mock_data/
│       └── users.go         # Hardcoded user data (10-20 items)
├── tests/
│   └── user_test.go         # Basic smoke tests
├── Containerfile
├── go.mod
└── go.sum

order-service/                     # Order Agent (Rust/Actix)
├── src/
│   ├── main.rs              # Actix-web entry point
│   ├── models.rs            # Data models
│   ├── handlers.rs          # HTTP handlers
│   └── mock_data.rs         # Hardcoded order data (10-20 items)
├── tests/
│   └── integration_tests.rs # Basic smoke tests
├── Containerfile
├── Cargo.toml
└── Cargo.lock

api-gateway/                       # Gateway Agent (Node.js/Express)
├── src/
│   ├── index.ts             # Express app entry point
│   ├── routes/
│   │   ├── products.ts      # Proxy to product-service
│   │   ├── users.ts         # Proxy to user-service
│   │   └── orders.ts        # Proxy to order-service
│   ├── middleware/
│   │   ├── cors.ts          # CORS configuration
│   │   ├── auth.ts          # JWT validation
│   │   └── rateLimit.ts     # Rate limiting
│   └── config.ts            # Service URLs configuration
├── tests/
│   └── gateway.test.ts
├── Containerfile
├── package.json
└── tsconfig.json

compose.yaml                       # Podman compose for all services
```

**Structure Decision**: Microservices architecture with polyglot services for demo purposes. Each service is independently developed, tested, and deployed by separate agents. Services return hardcoded mock data (no database, no external APIs). API Gateway aggregates responses and handles CORS. Frontend communicates only with gateway. Simple implementation to demonstrate multi-agent collaboration.

## Sequential Development Strategy

### Agent Structure & Work Sequence

**7 Agents** working in **sequential order**:

**Sequence 0: Project Setup** (All Agents - Collaborative)
- Define OpenAPI contracts for all services
- Agree on data models (Product, User, Order, pagination format)
- Configure upstream API URLs (environment variables per service)
- Initialize all service directories and dependencies

**Sequence 1: Backend Services** (⚡ Parallel Implementation)

1. **Product Agent** (Python/FastAPI) ⚡
   - **Scope**: Product service with mock data
   - **Deliverables**: GET /products endpoint, mock data (10-20 items), Containerfile
   - **Duration**: ~1-2 days
   - **Dependencies**: None (starts after Setup complete)
   - **Handoff**: Merges to master independently

2. **User Agent** (Go/Gin) ⚡
   - **Scope**: User service with mock data
   - **Deliverables**: GET /users endpoint, mock data (10-20 items), Containerfile
   - **Duration**: ~1-2 days
   - **Dependencies**: None (starts after Setup complete)
   - **Handoff**: Merges to master independently

3. **Order Agent** (Rust/Actix) ⚡
   - **Scope**: Order service with mock data
   - **Deliverables**: GET /orders endpoint, mock data with status badges, Containerfile
   - **Duration**: ~1-2 days
   - **Dependencies**: None (starts after Setup complete)
   - **Handoff**: Merges to master independently

⚡ **All 3 agents run in parallel** - Gateway Agent waits for all 3 to merge to master

**Sequence 2: Integration Layer** (Sequential Implementation)

4. **Gateway Agent** (Node.js/Express)
   - **Scope**: API Gateway, routing to all backend services, CORS
   - **Deliverables**: Proxy routes for products/users/orders, CORS middleware, Containerfile
   - **Duration**: ~1-2 days
   - **Handoff**: Complete before Frontend Agent starts

5. **Frontend Agent** (TypeScript/React)
   - **Scope**: Dashboard UI, all three sections (Products, Users, Orders)
   - **Deliverables**: ProductGrid, UserList, OrderTable, simple CSS, Containerfile
   - **Duration**: ~2-3 days
   - **Handoff**: Complete before DevOps Agent starts

**Sequence 3: Deployment**

6. **DevOps Agent** (Podman Compose)
   - **Scope**: Container orchestration, deployment configuration
   - **Deliverables**: compose.yaml, basic documentation
   - **Duration**: ~1 day
   - **Prerequisites**: ⚠️ **MUST WAIT FOR ALL 5 AGENTS** (Product, User, Order, Gateway, Frontend)
   - **Handoff**: Complete before QA Agent starts

**Sequence 4: Quality Assurance**

7. **QA Agent** (Testing & Validation)
   - **Scope**: Manual testing, smoke tests
   - **Deliverables**: Manual test checklist, verification that all 3 sections work
   - **Duration**: ~1 day
   - **Prerequisites**: ⚠️ **MUST WAIT FOR DevOps Agent ONLY** (deployed system running)
   - **Outcome**: Demo ready

### Communication Interfaces

**API Contracts** (defined in [contracts/](contracts/)):
- `api-gateway.yaml` - Gateway to Frontend contract
- `product-service.yaml` - Gateway to Product Service contract
- `user-service.yaml` - Gateway to User Service contract
- `order-service.yaml` - Gateway to Order Service contract

**Shared Data Models** (TypeScript types generated from OpenAPI):
- Frontend agent generates types from `api-gateway.yaml`
- Backend agents implement according to their service contract
- No shared code between services (only contract agreement)

### Independent Testing

Each agent can test independently:
- **Frontend Agent**: Manual browser testing
- **Product Agent**: Simple curl test or pytest basic smoke test
- **User Agent**: Simple curl test or go test basic smoke test
- **Order Agent**: Simple curl test or cargo test basic smoke test
- **Gateway Agent**: Manual curl tests for all routes

### Mock Data Structure

**Example mock data per service**:
```python
# product-service/app/mock_data.py
PRODUCTS = [
    {"id": 1, "name": "Widget", "price": 29.99},
    {"id": 2, "name": "Gadget", "price": 49.99},
    # ... 8-18 more items
]
```

Each agent hardcodes their own test data.

## Complexity Tracking

No constitution violations requiring justification.

**Note on Polyglot Architecture**: Using different languages per service adds complexity but enables:
- Sequential development by agents with different expertise
- Best tool for each job (Python for data, Go for performance, Rust for safety)
- Independent deployment and scaling
- Reduced merge conflicts (separate codebases)
- Clear handoff points between agents
