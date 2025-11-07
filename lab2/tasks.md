# Tasks: E-Commerce Dashboard (Demo - Mock Data)

**Input**: Design documents from `/specs/001-ecommerce-dashboard/`
**Prerequisites**: plan.md, spec.md, data-model.md, contracts/, research.md

**Architecture**: 5 independent services (Frontend, Product Service, User Service, Order Service, API Gateway) - all services use hardcoded mock data

**Organization**: Tasks organized in sequential workflow to demonstrate multi-agent collaboration

## Demo Purpose

**Goal**: Demonstrate 7 agents working sequentially on polyglot microservices
**Key Features**:
- Multi-language services (Python, Go, Rust, TypeScript)
- Sequential workflow with clear handoffs between agents
- Containerized deployment with podman-compose
- Simple mock data (no external APIs, no database)
- Each agent owns one service/responsibility

## Work Sequence

1. **Backend Services** (Sequence 1): ⚡ **Product Agent + User Agent + Order Agent** (run in parallel)
2. **Integration Layer** (Sequence 2): Gateway Agent → Frontend Agent (sequential)
3. **Deployment** (Sequence 3): DevOps Agent (waits for ALL above agents)
4. **Quality Assurance** (Sequence 4): QA Agent (waits for DevOps only)

## Format: `- [ ] [ID] [Story?] Description`

- **[Story]**: Which user story this task belongs to (US1, US2, US3)

## Path Conventions

**Multi-service structure**:
- **Frontend**: `frontend/src/`, `frontend/tests/`
- **Product Service**: `product-service/app/`
- **User Service**: `user-service/internal/`
- **Order Service**: `order-service/src/`
- **API Gateway**: `api-gateway/src/`
- **Shared**: `compose.yaml` at repository root

---

## Sequence 0: Project Setup

**Agent**: All Agents (Collaborative)
**Duration**: ~1-2 hours

### ⚠️ DEPENDENCY MANAGEMENT RULES (PREVENT MISSING MODULE ERRORS)

**READ THESE RULES BEFORE STARTING ANY SEQUENCE**

**Python (Product Service)**:
- ❌ **DON'T** use `EmailStr`, `HttpUrl`, `IPvAnyAddress` - they require extra packages
- ✅ **DO** use simple types: `str`, `int`, `float`, `bool`, `list`, `dict`
- ✅ **DO** include: `fastapi`, `uvicorn`, `pydantic` (base only)
- ⚠️ **IF** you must use `EmailStr`, add `email-validator` to requirements.txt
- ✅ **TEST** locally: `pip install -r requirements.txt && uvicorn app.main:app`

**Node.js (Gateway & Frontend)**:
- ❌ **DON'T** use `npm ci` (violates constitution - must use `npm install`)
- ✅ **DO** include `@types/*` for all TypeScript dependencies
- ✅ **DO** include: `@types/node`, `@types/express`, `@types/react`, `@types/react-dom`
- ✅ **DO** include PatternFly for frontend: `@patternfly/react-core`, `@patternfly/react-table`
- ✅ **TEST** locally: `npm install && npm run build`

**Go (User Service)**:
- ✅ **DO** run `go mod tidy` after adding any import
- ✅ **DO** run `go mod download` to verify all deps exist
- ✅ **DO** commit both `go.mod` and `go.sum`
- ✅ **TEST** locally: `go build ./cmd/server`

**Rust (Order Service)**:
- ✅ **DO** include feature flags: `serde = { version = "1.0", features = ["derive"] }`
- ✅ **DO** include tokio: `tokio = { version = "1", features = ["full"] }`
- ✅ **DO** commit `Cargo.lock`
- ✅ **TEST** locally: `cargo build`

### Directory Structure & Dependencies

- [ ] T001 Create product-service/ directory structure per plan.md
- [ ] T002 Create user-service/ directory structure per plan.md
- [ ] T003 Create order-service/ directory structure per plan.md
- [ ] T004 Create api-gateway/ directory structure per plan.md
- [ ] T005 Create frontend/ directory structure per plan.md
- [ ] T006 Initialize product-service/ with requirements.txt (fastapi==0.104.1, uvicorn==0.24.0, pydantic==2.5.0) - NO EmailStr or specialized types
- [ ] T007 Initialize user-service/ with go.mod (github.com/gin-gonic/gin v1.9.1) and run `go mod tidy`
- [ ] T008 Initialize order-service/ with Cargo.toml (actix-web="4", serde with derive feature, tokio with full features)
- [ ] T009 Initialize api-gateway/ with package.json (express, typescript, axios, @types/express, @types/node, cors, @types/cors) - ALWAYS use `npm install`
- [ ] T010 Initialize frontend/ with package.json (react, react-dom, typescript, axios, @patternfly/react-core, @patternfly/react-table, @types/react, @types/react-dom) - ALWAYS use `npm install`

### API Contracts

- [ ] T011 Create simple contracts/api-gateway.yaml OpenAPI spec (all teams review)

**Checkpoint**: Project structure ready - backend services can begin implementation

---

## Sequence 1: Backend Services Implementation

⚡ **PARALLEL EXECUTION**: Product, User, and Order agents run simultaneously (no dependencies between them)

### SEQUENCE 1A: Product Agent (Python/FastAPI) ⚡

**Agent**: Product Agent
**User Stories**: US1 - View Product Catalog Overview
**Duration**: ~1-2 days
**Dependencies**: None - starts immediately after Setup (T011) complete
**⚡ Can run in parallel with User Agent and Order Agent**

#### Pre-Flight Dependency Check

- [ ] T012 [US1] Verify product-service/requirements.txt exists with fastapi, uvicorn, pydantic (run `pip install -r requirements.txt` to test)

#### Product Service Implementation

- [ ] T013 [US1] Create product-service/app/main.py with FastAPI app setup
- [ ] T014 [US1] Create product-service/app/mock_data.py with 10-20 hardcoded products (id, name, price)
- [ ] T015 [US1] Create product-service/app/schemas.py with Product Pydantic model (use simple str types - NO EmailStr, NO HttpUrl)
- [ ] T016 [US1] Implement GET /products endpoint in product-service/app/routes.py (returns mock data)
- [ ] T017 [US1] Implement GET /health endpoint in product-service/app/routes.py

#### Product Service Testing & Containerization

- [ ] T018 [US1] Write basic pytest smoke test in product-service/tests/test_products.py
- [ ] T019 [US1] Create Containerfile for product-service (port 8001)
- [ ] T020 [US1] Test product service locally: uvicorn app.main:app --port 8001
- [ ] T020b [US1] Commit all product-service changes and merge product-agent branch to master
- [ ] T020c [US1] Verify merge successful: git log master --oneline -5 && ls product-service/Containerfile

**Checkpoint**: Product Service complete and MERGED to master

---

### SEQUENCE 1B: User Agent (Go/Gin) ⚡

**Agent**: User Agent
**User Stories**: US2 - Monitor User Accounts
**Duration**: ~1-2 days
**Dependencies**: None - starts immediately after Setup (T011) complete
**⚡ Can run in parallel with Product Agent and Order Agent**

#### Pre-Flight Dependency Check

- [ ] T021 [US2] Verify user-service/go.mod exists and run `go mod tidy && go mod download`

#### User Service Implementation

- [ ] T022 [US2] Create user-service/cmd/server/main.go with Gin setup
- [ ] T023 [US2] Create user-service/internal/mock_data/users.go with 10-20 hardcoded users (id, name, email)
- [ ] T024 [US2] Create user-service/internal/models/user.go with User struct
- [ ] T025 [US2] Implement GET /users handler in user-service/internal/handlers/user_handler.go (returns mock data)
- [ ] T026 [US2] Implement GET /health handler in user-service

#### User Service Testing & Containerization

- [ ] T027 [US2] Write basic go test smoke test in user-service/tests/user_test.go
- [ ] T028 [US2] Create Containerfile for user-service (port 8002)
- [ ] T029 [US2] Test user service locally: go run cmd/server/main.go
- [ ] T029b [US2] Commit all user-service changes and merge user-agent branch to master
- [ ] T029c [US2] Verify merge successful: git log master --oneline -5 && ls user-service/Containerfile

**Checkpoint**: User Service complete and MERGED to master

---

### SEQUENCE 1C: Order Agent (Rust/Actix) ⚡

**Agent**: Order Agent
**User Stories**: US3 - Track Order Status
**Duration**: ~1-2 days
**Dependencies**: None - starts immediately after Setup (T011) complete
**⚡ Can run in parallel with Product Agent and User Agent**

#### Order Service Implementation

- [ ] T036 [US3] Create order-service/src/main.rs with Actix-web setup
- [ ] T037 [US3] Create order-service/src/mock_data.rs with 10-20 hardcoded orders (id, customerId, status)
- [ ] T036 [US3] Create order-service/src/models.rs with Order struct and OrderStatus enum (Pending, Delivered, Cancelled)
- [ ] T037 [US3] Implement GET /orders handler in order-service/src/handlers.rs (returns mock data)
- [ ] T036 [US3] Implement GET /health handler in order-service/src/handlers.rs

#### Order Service Testing & Containerization

- [ ] T037 [US3] Write basic cargo test smoke test in order-service/tests/integration_tests.rs
- [ ] T037b [US3] Create Containerfile for order-service (port 8003)
- [ ] T037c [US3] Test order service locally: cargo run
- [ ] T037d [US3] Commit all order-service changes and merge order-agent branch to master
- [ ] T037e [US3] Verify merge successful: git log master --oneline -5 && ls order-service/Containerfile

**Checkpoint**: All backend services complete and MERGED to master - Gateway Agent can begin

---

## Sequence 2: Integration Layer

### SEQUENCE 2A: Gateway Agent (TypeScript/Express)

**Agent**: Gateway Agent
**User Stories**: US1, US2, US3
**Duration**: ~1-2 days
**Dependencies**: ⚠️ **MUST WAIT** for Product (T020c) + User (T029c) + Order (T037e) all merged to master

#### Gateway Foundation & Middleware

- [ ] T036 Create api-gateway/src/index.ts with Express app setup
- [ ] T037 Implement CORS middleware in api-gateway/src/middleware/cors.ts (allow origin: http://localhost:3001)
- [ ] T038 Create service configuration in api-gateway/src/config.ts (product:8001, user:8002, order:8003)

#### Gateway Routing

- [ ] T039 [US1] Implement GET /products route in api-gateway/src/routes/products.ts (proxy to product-service:8001)
- [ ] T040 [US2] Implement GET /users route in api-gateway/src/routes/users.ts (proxy to user-service:8002)
- [ ] T041 [US3] Implement GET /orders route in api-gateway/src/routes/orders.ts (proxy to order-service:8003)
- [ ] T042 Implement GET /health route in api-gateway

#### Gateway Testing & Containerization

- [ ] T043 Test gateway routes manually with curl (verify all 3 endpoints work)
- [ ] T044 Create Containerfile for api-gateway (port 8000)
- [ ] T045 Verify CORS headers in responses (should include Access-Control-Allow-Origin: http://localhost:3001)
- [ ] T045b Commit all api-gateway changes and merge gateway-agent branch to master
- [ ] T045c Verify merge successful: git log master --oneline -5 && ls api-gateway/Containerfile

**Checkpoint**: API Gateway complete and MERGED to master - Frontend Agent can begin

---

### SEQUENCE 2B: Frontend Agent (TypeScript/React + PatternFly)

**Agent**: Frontend Agent
**User Stories**: US1 (Products), US2 (Users), US3 (Orders)
**Duration**: ~2-3 days
**Total Tasks**: 27 (T046-T070c)

#### 🔑 Critical Requirements

| Requirement     | Detail                                      |
|-----------------|---------------------------------------------|
| Framework       | React 18+ with TypeScript + Vite           |
| UI Library      | PatternFly 5 components                     |
| Port            | 3001 (frontend)                             |
| API Gateway     | http://localhost:8000                       |
| Package Manager | npm install (never npm ci)                  |
| Container       | Containerfile with port 3001                |
| Sections        | 3 views: Grid (Products), List (Users), Table (Orders) |
| Status Colors   | Yellow (Pending), Green (Delivered), Gray (Cancelled)   |

#### 1. Foundation (T046-T049)

- [ ] T046 Create frontend/src/main.tsx and frontend/src/App.tsx with React 18+ setup and PatternFly CSS imports
  ```typescript
  // main.tsx must import PatternFly CSS
  import '@patternfly/react-core/dist/styles/base.css';
  ```
- [ ] T047 Setup Vite config in frontend/vite.config.ts (port 3001, proxy to gateway:8000)
- [ ] T048 Create frontend/src/services/api.ts (axios client with base URL http://localhost:8000)
- [ ] T049 Create frontend/src/components/layout/Dashboard.tsx using PatternFly Page component (container for 3 sections)

#### 2. Products Section - Grid View (T050-T054) [US1]

- [ ] T050 [US1] Create frontend/src/services/products.ts (API call to GET /products)
- [ ] T051 [US1] Create frontend/src/components/products/ProductCard.tsx using PatternFly Card component
  - Display: product name and price
  - Use Card, CardTitle, CardBody from @patternfly/react-core
- [ ] T052 [US1] Create frontend/src/components/products/ProductGrid.tsx using PatternFly Gallery component
  - Use Gallery component for responsive grid layout
  - Renders multiple ProductCard components
- [ ] T053 [US1] Add custom CSS overrides if needed in frontend/src/styles/dashboard.css (PatternFly provides base styling)
- [ ] T054 [US1] Integrate ProductGrid into Dashboard layout

#### 3. Users Section - List View (T055-T059) [US2]

- [ ] T055 [US2] Create frontend/src/services/users.ts (API call to GET /users)
- [ ] T056 [US2] Create frontend/src/components/users/UserListItem.tsx using PatternFly List/ListItem components
  - Display: user name and email
  - Use List, ListItem from @patternfly/react-core
- [ ] T057 [US2] Create frontend/src/components/users/UserList.tsx using PatternFly List component
  - Renders multiple UserListItem components
- [ ] T058 [US2] Add custom CSS overrides if needed in frontend/src/styles/dashboard.css (PatternFly provides base styling)
- [ ] T059 [US2] Integrate UserList into Dashboard layout

#### 4. Orders Section - Table View (T060-T065) [US3]

- [ ] T060 [US3] Create frontend/src/services/orders.ts (API call to GET /orders)
- [ ] T061 [US3] Create frontend/src/components/orders/StatusBadge.tsx using PatternFly Label component
  - Color coding: Yellow (Pending), Green (Delivered), Gray (Cancelled)
  - Use Label component with color prop from @patternfly/react-core
- [ ] T062 [US3] Create frontend/src/components/orders/OrderRow.tsx (table row with StatusBadge)
- [ ] T063 [US3] Create frontend/src/components/orders/OrderTable.tsx using PatternFly Table component
  - Use Table, Thead, Tbody, Tr, Th, Td from @patternfly/react-table
  - Columns: Order ID, Customer, Status
- [ ] T064 [US3] Add custom CSS overrides if needed in frontend/src/styles/dashboard.css (PatternFly provides base styling)
- [ ] T065 [US3] Integrate OrderTable into Dashboard layout

#### 5. Common Components (T066-T067)

- [ ] T066 Create frontend/src/components/common/LoadingSpinner.tsx using PatternFly Spinner component
- [ ] T067 Create frontend/src/components/common/ErrorMessage.tsx using PatternFly Alert component (danger variant)

#### 6. Testing & Containerization (T068-T070c)

- [ ] T068 Test dashboard manually in browser at http://localhost:3001
  - Verify all 3 sections display simultaneously
  - Verify Products grid shows name and price
  - Verify Users list shows name and email
  - Verify Orders table shows status badges with correct colors
- [ ] T069 Create Containerfile for frontend (port 3001)
- [ ] T070 Verify frontend uses npm install (not npm ci) in Containerfile
- [ ] T070b Commit all frontend changes and merge frontend-agent branch to master
- [ ] T070c Verify merge successful: git log master --oneline -5 && ls frontend/Containerfile

**Checkpoint**: ✅ Frontend complete and MERGED to master - Pre-deployment verification required before DevOps begins

---

## Sequence 3: Deployment

### SEQUENCE 3: DevOps Agent (Podman Compose)

**Agent**: DevOps Agent
**Duration**: ~1 day

#### Pre-Deployment Verification (REQUIRED)

- [ ] T071 ⚠️ CRITICAL: Verify ALL 5 services exist in master branch before starting DevOps work:
  ```bash
  git checkout master
  git pull
  ls product-service/Containerfile
  ls user-service/Containerfile
  ls order-service/Containerfile
  ls api-gateway/Containerfile
  ls frontend/Containerfile
  # ALL 5 must exist - if any missing, STOP and wait for that agent to complete
  ```

#### Compose Configuration

- [ ] T072 Create compose.yaml at repository root with all 5 services defined
- [ ] T073 Add product-service to compose.yaml (port 8001, restart: unless-stopped)
- [ ] T074 Add user-service to compose.yaml (port 8002, restart: unless-stopped)
- [ ] T075 Add order-service to compose.yaml (port 8003, restart: unless-stopped)
- [ ] T076 Add api-gateway to compose.yaml (port 8000, restart: unless-stopped, depends_on: backend services)
- [ ] T077 Add frontend to compose.yaml (port 3001, restart: unless-stopped, depends_on: api-gateway)

#### Container Build & Validation

- [ ] T078 Build all containers: podman-compose build
- [ ] T079 Verify all services use podman-compose (NOT docker-compose)
- [ ] T080 Start all services: podman-compose up -d
- [ ] T081 Verify all services running: podman ps
- [ ] T082 Check service health: curl http://localhost:8001/health (product)
- [ ] T083 Check service health: curl http://localhost:8002/health (user)
- [ ] T084 Check service health: curl http://localhost:8003/health (order)
- [ ] T085 Check gateway health: curl http://localhost:8000/health
- [ ] T086 Verify correct ports: frontend (3001), gateway (8000), services (8001-8003)

#### Documentation

- [ ] T087 Create README.md in repository root with quickstart instructions
- [ ] T088 Add service-specific README.md files with build/run instructions
- [ ] T089 Commit all DevOps changes and merge devops-agent branch to master
- [ ] T090 Verify merge successful: git log master --oneline -5 && ls compose.yaml

**Checkpoint**: All services deployed, healthy, and MERGED to master - QA Agent can begin testing

---

## Sequence 4: Quality Assurance

### SEQUENCE 4: QA Agent (Manual Testing)

**Agent**: QA Agent
**Duration**: ~1 day

#### Pre-QA Verification (REQUIRED)

- [ ] T091 ⚠️ CRITICAL: Verify compose.yaml exists in master branch and system is deployed:
  ```bash
  git checkout master
  git pull
  ls compose.yaml
  podman ps
  # Must show 5 containers running: product-service, user-service, order-service, api-gateway, frontend
  # If not running, STOP and wait for DevOps to complete
  ```

#### API Gateway Validation

- [ ] T092 Verify GET http://localhost:8000/products returns product data
- [ ] T093 Verify GET http://localhost:8000/users returns user data
- [ ] T094 Verify GET http://localhost:8000/orders returns order data
- [ ] T095 Verify CORS headers present in all API responses (Access-Control-Allow-Origin: http://localhost:3001)

#### Frontend Validation

- [ ] T096 [US1] Load http://localhost:3001 and verify Products section displays in grid view
- [ ] T097 [US1] Verify product cards show name and price
- [ ] T098 [US1] Verify 10-20 products are visible
- [ ] T099 [US2] Verify Users section displays in list view
- [ ] T100 [US2] Verify user list items show name and email
- [ ] T101 [US2] Verify 10-20 users are visible
- [ ] T102 [US3] Verify Orders section displays in table view
- [ ] T103 [US3] Verify order rows show status badges with correct colors (yellow/green/gray)
- [ ] T104 [US3] Verify 10-20 orders are visible
- [ ] T105 Verify all 3 sections (Products, Users, Orders) visible simultaneously on dashboard

#### Service Independence Validation

- [ ] T106 Stop product-service and verify error message appears in Products section (other sections still work)
- [ ] T107 Restart product-service and verify Products section works again
- [ ] T108 Verify each service can be restarted independently: podman-compose restart <service>

#### Final Validation

- [ ] T109 Verify complete system restart: podman-compose down && podman-compose up -d
- [ ] T110 Create manual test checklist document with pass/fail results
- [ ] T111 Commit QA results and merge qa-agent branch to master
- [ ] T112 Verify merge successful: git log master --oneline -5

**Checkpoint**: All quality checks passed and MERGED to master - Demo ready ✅

---

---

## Dependencies & Agent Handoffs

### Visual Dependency Chain

```
Sequence 0: Setup (All Agents Collaborate)
    ↓
    T011 Complete (API Contracts defined)
    ↓
    ⚡ PARALLEL EXECUTION START ⚡
    ↓
    ├─→ Sequence 1A: Product Agent (product-agent branch)
    │       ↓
    │       T020c Complete (Product Service MERGED to master)
    │
    ├─→ Sequence 1B: User Agent (user-agent branch)
    │       ↓
    │       T029c Complete (User Service MERGED to master)
    │
    └─→ Sequence 1C: Order Agent (order-agent branch)
            ↓
            T037e Complete (Order Service MERGED to master)
    ↓
    ⚡ PARALLEL EXECUTION END - ALL 3 must complete ⚡
    ↓
    ALL 3 Backend Services MERGED to master
    ↓
Sequence 2A: Gateway Agent (gateway-agent branch)
    ↓
    T045c Complete (Gateway MERGED to master)
    ↓
Sequence 2B: Frontend Agent (frontend-agent branch)
    ↓
    T070c Complete (Frontend MERGED to master)
    ↓
    ⚠️ WAIT: All 5 services must be MERGED to master ⚠️
    (Product + User + Order + Gateway + Frontend in master branch)
    ↓
Sequence 3: DevOps Agent (devops-agent branch, WORKS FROM master)
    ↓
    T071: ⚠️ VERIFY all 5 services exist in master
    ↓
    T078-T086: Build and deploy all services
    ↓
    T090 Complete (compose.yaml MERGED to master)
    ↓
    ⚠️ WAIT: Deployment must be complete and MERGED ⚠️
    ↓
Sequence 4: QA Agent (qa-agent branch, WORKS FROM master)
    ↓
    T091: ⚠️ VERIFY deployment complete in master
    ↓
    T092-T110: Test all services
    ↓
    T112 Complete (QA results MERGED to master - Demo validated and ready)
```

### Blocking Dependencies by Task

| Task Range | Agent | Blocked By | Can Start After | Parallel? |
|------------|-------|------------|-----------------|-----------|
| T001-T011 | All Agents | None | Immediately | Collaborative |
| T012-T020c | Product Agent ⚡ | T011 (API contracts) | Setup complete | **YES - with User & Order** |
| T021-T029c | User Agent ⚡ | T011 (API contracts) | Setup complete | **YES - with Product & Order** |
| T030-T037e | Order Agent ⚡ | T011 (API contracts) | Setup complete | **YES - with Product & User** |
| T038-T045c | Gateway Agent | **T020c + T029c + T037e** (All 3 backends merged) | **ALL 3 backends merged** | No |
| T046-T070c | Frontend Agent | T045c (Gateway merged to master) | Gateway Agent merged | No |
| T071-T090 | DevOps Agent | **T020c + T029c + T037e + T045c + T070c** (ALL 5 merged to master) | **ALL agents merged** | No |
| T091-T112 | QA Agent | T090 (DevOps merged to master) | **DevOps Agent merged** | No |

### Critical Path Dependencies

**Hard Blockers** (agent cannot start until previous completes):

1. **Product Agent** ⚡ depends on:
   - ✅ T011: API contracts defined (Setup complete)
   - ⚡ **CAN RUN IN PARALLEL** with User and Order agents

2. **User Agent** ⚡ depends on:
   - ✅ T011: API contracts defined (Setup complete)
   - ⚡ **CAN RUN IN PARALLEL** with Product and Order agents

3. **Order Agent** ⚡ depends on:
   - ✅ T011: API contracts defined (Setup complete)
   - ⚡ **CAN RUN IN PARALLEL** with Product and User agents

4. **Gateway Agent** depends on:
   - ✅ T020c: Product service in master (port 8001)
   - ✅ T029c: User service in master (port 8002)
   - ✅ T037e: Order service in master (port 8003)
   - ⚠️ **CRITICAL**: All 3 backend services must be merged to master
   - ⚠️ **BLOCKS until ALL 3 parallel agents complete**

5. **Frontend Agent** depends on:
   - ✅ T045c: Gateway merged to master (port 8000)
   - ✅ CORS headers configured
   - ⚠️ **CRITICAL**: Cannot build UI without working API

6. **DevOps Agent** depends on:
   - ✅ T020c: Product service merged to master with Containerfile
   - ✅ T029c: User service merged to master with Containerfile
   - ✅ T037e: Order service merged to master with Containerfile
   - ✅ T045c: Gateway merged to master with Containerfile
   - ✅ T070c: Frontend merged to master with Containerfile
   - ⚠️ **CRITICAL**: ALL 5 SERVICES must be merged to master before DevOps starts
   - ⚠️ **CRITICAL**: DevOps works from master branch and creates compose.yaml

7. **QA Agent** depends on:
   - ✅ T090: All services deployed via podman-compose and merged to master
   - ⚠️ **CRITICAL**: Complete deployed system must be running from master branch
   - ⚠️ **CRITICAL**: QA cannot start until DevOps completes deployment and merge

### Handoff Criteria

**Each agent must complete these before handoff**:

| Agent | Handoff Task | Acceptance Criteria | Next Agent |
|-------|--------------|---------------------|------------|
| **Setup** | T011 | contracts/api-gateway.yaml exists with all 3 endpoints defined | ⚡ Product + User + Order (parallel) |
| **Product Agent** ⚡ | T020c | Product service merged to master, Containerfile exists in master | (works in parallel) |
| **User Agent** ⚡ | T029c | User service merged to master, Containerfile exists in master | (works in parallel) |
| **Order Agent** ⚡ | T037e | Order service merged to master, Containerfile exists in master | (works in parallel) |
| **All 3 Backend Agents** | T020c + T029c + T037e | All 3 services merged to master | Gateway Agent |
| **Gateway Agent** | T045c | Gateway merged to master, Containerfile exists in master | Frontend Agent |
| **Frontend Agent** | T070c | Frontend merged to master, Containerfile exists in master | ⚠️ **DevOps Agent (waits for ALL 5 in master)** |
| **DevOps Agent** | T090 | compose.yaml merged to master, all 5 services deployed successfully | ⚠️ **QA Agent (waits for DevOps)** |
| **QA Agent** | T112 | Manual test checklist 100% pass, results merged to master | ✅ **Demo Complete** |

### Agent Work Notes

**Each agent works independently** on their assigned tasks:

**⚡ PARALLEL AGENTS** - Run simultaneously:

**Product Agent** (T012-T020c) ⚡:
- Single agent completes all tasks including merge to master
- **Runs in parallel with User and Order agents**
- Must merge to master before marking work complete

**User Agent** (T021-T029c) ⚡:
- Single agent completes all tasks including merge to master
- **Runs in parallel with Product and Order agents**
- Must merge to master before marking work complete

**Order Agent** (T030-T037e) ⚡:
- Single agent completes all tasks including merge to master
- **Runs in parallel with Product and User agents**
- Must merge to master before marking work complete

**SEQUENTIAL AGENTS** - Wait for previous agents:

**Gateway Agent** (T038-T045c):
- Single agent completes all tasks including merge to master
- Must merge to master before marking work complete

**Frontend Agent** (T046-T070c):
- Single agent completes all tasks including merge to master
- Must merge to master before marking work complete

**DevOps Agent** (T071-T090):
- Single agent works from master branch (requires all 5 services merged)
- Must verify all services exist before starting
- Must merge compose.yaml to master before marking work complete

**QA Agent** (T091-T112):
- Single agent works from master branch (requires DevOps merged)
- Must verify deployment complete before starting
- Must merge QA results to master before marking work complete

### Risk: Cascading Delays

**Warning**: Sequential workflow means delays cascade:
- Product Agent +1 day → User Agent starts 1 day late → Order Agent starts 2 days late → etc.
- **Mitigation**: Strict handoff criteria prevent incomplete work from blocking downstream agents

### Special Synchronization Points

**⚠️ CRITICAL: DevOps Agent Synchronization Point**:
- DevOps Agent MUST wait for completion of **ALL 5 AGENTS**:
  - ✅ Product Agent (T020c complete - MERGED to master)
  - ✅ User Agent (T029c complete - MERGED to master)
  - ✅ Order Agent (T037e complete - MERGED to master)
  - ✅ Gateway Agent (T045c complete - MERGED to master)
  - ✅ Frontend Agent (T070c complete - MERGED to master)
- **Reason**: DevOps needs ALL 5 Containerfiles to exist in master branch before creating compose.yaml
- **Verification Before Starting T071**:
  ```bash
  git checkout master
  git pull
  ls product-service/Containerfile
  ls user-service/Containerfile
  ls order-service/Containerfile
  ls api-gateway/Containerfile
  ls frontend/Containerfile
  # All 5 must exist in master branch - if any missing, STOP and wait for that agent to merge
  ```

**⚠️ CRITICAL: QA Agent Synchronization Point**:
- QA Agent MUST wait for **DevOps Agent ONLY**:
  - ✅ DevOps Agent (T090 complete - MERGED to master)
- **Reason**: QA needs the complete deployed system running from master branch
- **Verification Before Starting T091**:
  ```bash
  git checkout master
  git pull
  ls compose.yaml
  podman ps
  # Must show 5 containers running:
  # product-service, user-service, order-service, api-gateway, frontend
  ```

---

## Summary

**Total Tasks**: 112 (includes merge tasks for each agent)
**Organized by Sequence**:
- Sequence 0 (Setup): 11 tasks (~1-2 hours)
- Sequence 1A (Product Agent): 10 tasks including merge (~1-2 days)
- Sequence 1B (User Agent): 10 tasks including merge (~1-2 days)
- Sequence 1C (Order Agent): 10 tasks including merge (~1-2 days)
- Sequence 2A (Gateway Agent): 10 tasks including merge (~1-2 days)
- Sequence 2B (Frontend Agent): 27 tasks including merge (~2-3 days)
- Sequence 3 (DevOps Agent): 20 tasks including verification and merge (~1 day)
- Sequence 4 (QA Agent): 22 tasks including verification and merge (~1 day)

**User Story Coverage**:
- US1 (Products): 13 tasks
- US2 (Users): 13 tasks
- US3 (Orders): 13 tasks

**Execution Flow (7 Agents - Parallel + Sequential with Git Integration)**:
1. All agents collaborate on Setup (T001-T011)
2. ⚡ **PARALLEL**: Product, User, Order agents build services simultaneously, merge to master (T012-T037e)
3. Gateway Agent integrates all backends (waits for all 3), merges to master (T038-T045c)
4. Frontend Agent builds dashboard UI (waits for Gateway), merges to master (T046-T070c)
5. DevOps Agent verifies master (waits for ALL 5), deploys with podman-compose, merges to master (T071-T090)
6. QA Agent verifies deployment (waits for DevOps), validates system, merges results to master (T091-T112)

**Key Simplifications from Original**:
- ✅ Mock data instead of upstream APIs (removed 40+ HTTP client/cache/pagination tasks)
- ✅ Basic smoke tests instead of comprehensive testing (removed 30+ test tasks)
- ✅ Simple implementation (removed complex features like cursor pagination, caching, performance optimization)
- ✅ Manual testing instead of E2E automation (removed Playwright complexity)
- ✅ 3 status badges instead of 5 (simplified order status)
- ✅ No images, no avatars, no complex formatting

**Demo Duration**: ~8-10 days with parallel backend execution (vs 10-12 days fully sequential, vs 25-35 days original)

**Key Principles**:
- ⚡ **Parallel execution** for independent backend services (Product, User, Order)
- **Sequential execution** for dependent services (Gateway waits for backends, Frontend waits for Gateway)
- Each agent completes their work and merges to master before dependent agents start
- Polyglot microservices (Python, Go, Rust, TypeScript)
- Containerized deployment (Podman-only)
- Simple mock data (no external dependencies)

---

## Common Issues & Troubleshooting

### Product Service (Python/FastAPI)

**Issue**: `ModuleNotFoundError: No module named 'email_validator'`
```
ImportError: email-validator is not installed, run `pip install 'pydantic[email]'`
```

**Root Cause**: Pydantic's `EmailStr` type requires the `email-validator` package

**Solutions**:
1. **Option A (Recommended for Demo)**: Avoid `EmailStr` in schemas
   ```python
   # Instead of:
   email: EmailStr

   # Use:
   email: str
   ```

2. **Option B**: Add dependency to requirements.txt
   ```
   pydantic[email]==2.5.0
   ```
   Or explicitly:
   ```
   email-validator==2.1.0
   ```

3. **Option C**: Rebuild container after updating requirements.txt
   ```bash
   podman-compose build product-service
   podman-compose up -d product-service
   ```

**Prevention**: Keep schemas simple for demo - avoid specialized Pydantic types that require extra dependencies

---

### User Service (Go/Gin)

**Issue**: `go: module not found`

**Solution**:
```bash
cd user-service
go mod tidy
go mod download
```

---

### Order Service (Rust/Actix)

**Issue**: Long compile times

**Solution**: Use release mode only for final build
```bash
cargo build  # Debug mode (faster compile)
cargo run    # Debug mode
```

---

### API Gateway (Node.js/Express)

**Issue**: `CORS error in browser console`

**Solution**: Verify CORS middleware includes correct origin
```typescript
// api-gateway/src/middleware/cors.ts
origin: 'http://localhost:3001'  // Must match frontend port
```

---

### Frontend (React/Vite)

**Issue**: `npm ci` used instead of `npm install`

**Solution**: Per constitution, ALWAYS use `npm install`
```bash
npm install  # Correct
npm ci       # Wrong - violates constitution
```

---

### DevOps (Podman Compose)

**Issue**: `docker-compose` command not found

**Solution**: Must use `podman-compose` per constitution
```bash
podman-compose up -d   # Correct
docker-compose up -d   # Wrong - violates constitution
```

**Issue**: Services not starting

**Solution**: Check health endpoints
```bash
podman ps  # Check running containers
curl http://localhost:8001/health  # Product
curl http://localhost:8002/health  # User
curl http://localhost:8003/health  # Order
curl http://localhost:8000/health  # Gateway
```

---

### General Debugging

**Check logs**:
```bash
podman-compose logs product-service
podman-compose logs user-service
podman-compose logs order-service
podman-compose logs api-gateway
podman-compose logs frontend
```

**Restart individual service**:
```bash
podman-compose restart product-service
```

**Full rebuild**:
```bash
podman-compose down
podman-compose build
podman-compose up -d
```
