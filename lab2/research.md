# Research: E-Commerce Dashboard

**Feature**: 001-ecommerce-dashboard
**Date**: 2025-10-28
**Purpose**: Resolve technical unknowns and select technologies for dashboard implementation

## Research Questions

1. Frontend framework selection (React/Vue/Svelte)
2. Test framework selection (Jest/Vitest for unit, Playwright/Cypress for E2E)
3. API client library selection
4. Pagination strategy for large datasets
5. Caching strategy for API responses

---

## 1. Frontend Framework Selection

### Decision: **React 18.2+**

### Rationale

- **Ecosystem maturity**: Largest component library ecosystem (Material-UI, shadcn/ui, Ant Design) provides pre-built grid, list, and table components matching requirements
- **Performance**: React 18 concurrent features and automatic batching improve rendering of large data lists (1000+ orders)
- **Team familiarity**: React is most common in e-commerce space, maximizing developer availability
- **TypeScript support**: First-class TypeScript support for type-safe API contracts
- **Testing ecosystem**: Mature testing libraries (React Testing Library, Vitest, Playwright) align with constitution requirements

### Alternatives Considered

- **Vue 3**: Excellent performance and simpler learning curve, but smaller component ecosystem for admin dashboards
- **Svelte**: Best raw performance and smallest bundle size, but immature admin UI library ecosystem and smaller talent pool
- **Solid.js**: Superior fine-grained reactivity, but too new for enterprise use (lack of mature libraries, small community)

### Implementation Notes

- Use Vite as build tool (faster than webpack, official React support)
- Use React Router for client-side routing (if needed for future expansion)
- Prefer functional components with hooks (no class components)

---

## 2. Test Framework Selection

### Decision: **Vitest + Playwright**

### Rationale

**Unit/Integration Tests - Vitest**:
- **Vite native**: Zero-config integration with Vite build tool
- **Performance**: 10x faster than Jest (HMR-based test execution, native ESM)
- **Jest compatibility**: Drop-in replacement for Jest API (easy migration if needed)
- **Coverage**: Built-in coverage via c8/Istanbul

**E2E Tests - Playwright**:
- **Cross-browser**: Tests against Chromium, Firefox, WebKit (Safari) matching constitution's browser support requirement
- **Reliability**: Auto-wait and retry mechanisms reduce flaky tests
- **Modern API**: Async/await native, better DX than Cypress
- **Network mocking**: Built-in request interception for API testing without backend

### Alternatives Considered

- **Jest**: Industry standard but slower than Vitest, requires additional config for Vite/ESM
- **Cypress**: Popular but Electron-based (heavier), no native multi-browser support, commercial features behind paywall
- **Testing Library**: Not a framework but a library - will use `@testing-library/react` WITH Vitest for component tests

### Implementation Notes

- Vitest for component tests (`tests/components/`) and API integration tests (`tests/integration/`)
- Playwright for E2E flows (`tests/e2e/`)
- Minimum coverage: >80% for services layer (API calls), >60% for components (visual focus)

---

## 3. API Client Library

### Decision: **Axios**

### Rationale

- **Interceptors**: Request/response interceptors enable centralized error handling, auth token injection, and CORS header verification (constitution requirement)
- **Timeout handling**: Built-in timeout configuration for <2s API response requirement
- **Request cancellation**: AbortController support for canceling in-flight requests on component unmount
- **Browser support**: Works in all target browsers (last 2 versions of major browsers)
- **Error handling**: Automatic JSON parsing and consistent error structure
- **Mock-friendly**: Easy to mock in tests with axios-mock-adapter

### Alternatives Considered

- **Fetch API**: Native but lacks interceptors, timeout handling requires manual AbortController, less consistent error handling
- **ky**: Modern fetch wrapper but smaller ecosystem, less battle-tested for enterprise apps
- **TanStack Query (React Query)**: Excellent for caching/state management but overkill for simple dashboard; adds complexity without clear benefit given requirements

### Implementation Notes

- Create single axios instance with base URL configuration
- Add interceptors for error handling (network errors, 4xx/5xx responses)
- Add interceptor to verify CORS headers in development (console.log Access-Control-Allow-Origin)
- Configure default timeout: 2000ms (matching performance goal)

---

## 4. Pagination Strategy

### Decision: **Cursor-based pagination with page size limits**

### Rationale

- **Performance**: Cursor-based avoids offset/limit performance issues with large datasets (1000+ orders)
- **Consistency**: Prevents duplicate/missing items when data changes during pagination
- **API gateway compatibility**: Most modern APIs support cursor pagination
- **Flexibility**: Falls back to offset pagination if API doesn't support cursors

### Implementation Approach

```javascript
// API contract for paginated endpoints
GET /api/products?limit=50&cursor=eyJpZCI6MTIzfQ==
Response: {
  data: [...],
  pagination: {
    nextCursor: "eyJpZCI6MTczfQ==",
    hasMore: true
  }
}
```

**Page size limits** (matching constitution performance requirements):
- Products: 50 items per page (grid view loads quickly)
- Users: 100 items per page (list view is lighter weight)
- Orders: 50 items per page (table with multiple columns)

### Alternatives Considered

- **Offset pagination**: Simpler but degrades with large offsets (OFFSET 10000 is slow)
- **Infinite scroll**: Good UX but harder to test, users may want "jump to page" functionality
- **Load all data**: Violates performance requirements (1000+ orders would exceed 3s load time)

### Implementation Notes

- Use URL query params to preserve pagination state (back button support)
- Show "Load More" button + pagination controls for flexibility
- Cache current page in memory (no backend calls when switching between sections)

---

## 5. Caching Strategy

### Decision: **In-memory cache with stale-while-revalidate + manual refresh**

### Rationale

- **Simplicity**: Meets "manual refresh" requirement from spec assumptions (no complex cache invalidation)
- **Performance**: Cached responses improve perceived performance on section switching
- **Fresh data**: Manual refresh button gives users control over data freshness
- **Browser caching**: Use Cache-Control headers from API gateway when available

### Implementation Approach

```javascript
// Simple in-memory cache
const cache = new Map();
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

async function fetchWithCache(url) {
  const cached = cache.get(url);
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data; // Return stale data immediately
  }

  const data = await axios.get(url);
  cache.set(url, { data, timestamp: Date.now() });
  return data;
}
```

**Cache invalidation**:
- Manual refresh button clears cache for current section
- Global refresh clears entire cache
- 5-minute TTL auto-invalidates stale data

### Alternatives Considered

- **No caching**: Simpler but wastes API calls when switching between sections
- **LocalStorage**: Persists across sessions but adds complexity, potential staleness issues
- **Service Worker**: Overkill for admin dashboard, adds deployment complexity
- **TanStack Query**: Sophisticated caching but adds library weight for minimal benefit

### Implementation Notes

- Cache only successful responses (200 status)
- Show "Last updated: X minutes ago" timestamp in UI
- Add cache warming on dashboard load (parallel fetch all three sections)

---

## 6. Multi-Language Service Architecture

### Decision: **Polyglot microservices for parallel development**

### Rationale

- **Team Autonomy**: Different teams can work independently without merge conflicts
- **Best Tool for Job**: Python (FastAPI) for rapid API development, Go for high performance, Rust for memory safety, TypeScript for full-stack consistency
- **Parallel Development**: 5 teams can work simultaneously on different services
- **Independent Deployment**: Each service can be deployed/scaled independently
- **Skill Matching**: Teams work in their language of expertise

### Service Language Assignments

1. **Frontend** → TypeScript/React (UI expertise, type safety)
2. **Product Service** → Python/FastAPI (rapid development, rich ecosystem)
3. **User Service** → Go/Gin (performance, concurrency for auth checks)
4. **Order Service** → Rust/Actix (data integrity critical for orders)
5. **API Gateway** → TypeScript/Express (consistent with frontend, Node.js ecosystem)

### Alternatives Considered

- **Monolingual (all TypeScript)**: Simpler but limits team autonomy, single point of bottleneck
- **2 languages (TypeScript + Python)**: Reduces variety but still requires coordination between frontend/backend teams
- **All separate repos**: More isolation but harder contract management

---

## Technology Stack Summary

### Frontend (TypeScript/React)
| Component | Technology | Version |
|-----------|-----------|---------|
| Language | TypeScript | 5.x |
| Framework | React | 18.2+ |
| Build Tool | Vite | 5.x |
| API Client | Axios | 1.6+ |
| Unit/Component Tests | Vitest | 1.x |
| E2E Tests | Playwright | 1.x |
| Runtime | Node.js LTS | 20.x |

### Product Service (Python)
| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Python | 3.11+ |
| Framework | FastAPI | 0.104+ |
| ORM | SQLAlchemy | 2.0+ |
| Validation | Pydantic | 2.x |
| Testing | pytest | 7.x+ |
| ASGI Server | Uvicorn | 0.24+ |

### User Service (Go)
| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Go | 1.21+ |
| Framework | Gin | 1.9+ |
| Database | pgx (PostgreSQL) | 5.x |
| Testing | go test | (stdlib) |
| Migration | golang-migrate | 4.x |

### Order Service (Rust)
| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Rust | 1.75+ |
| Framework | Actix-web | 4.x |
| Database | sqlx | 0.7+ |
| Serialization | Serde | 1.x |
| Testing | cargo test | (built-in) |

### API Gateway (Node.js)
| Component | Technology | Version |
|-----------|-----------|---------|
| Language | TypeScript | 5.x |
| Framework | Express | 4.x |
| HTTP Client | Axios | 1.6+ |
| Testing | Vitest + Supertest | 1.x + 6.x |
| Runtime | Node.js LTS | 20.x |

### Shared Infrastructure
| Component | Technology | Version |
|-----------|-----------|---------|
| Database | PostgreSQL | 15+ |
| Container | Podman | 4.x+ |
| Orchestration | podman-compose | 1.x |

---

## Dependencies by Service

### Frontend (package.json)
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0",
    "typescript": "^5.3.0",
    "vitest": "^1.0.0",
    "@testing-library/react": "^14.0.0",
    "@playwright/test": "^1.40.0",
    "axios-mock-adapter": "^1.22.0"
  }
}
```

### Product Service (requirements.txt)
```
fastapi==0.104.1
uvicorn==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
pydantic==2.5.0
pytest==7.4.3
httpx==0.25.2
```

### User Service (go.mod)
```go
module github.com/example/user-service

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/jackc/pgx/v5 v5.5.0
    github.com/golang-migrate/migrate/v4 v4.16.2
)
```

### Order Service (Cargo.toml)
```toml
[dependencies]
actix-web = "4.4"
sqlx = { version = "0.7", features = ["postgres", "runtime-tokio-rustls"] }
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.35", features = ["full"] }
```

### API Gateway (package.json)
```json
{
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.6.0",
    "cors": "^2.8.5",
    "express-rate-limit": "^7.1.5"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "vitest": "^1.0.0",
    "supertest": "^6.3.3"
  }
}
```

---

## Next Steps (Phase 1)

1. ✅ Framework decisions finalized - proceed to data modeling
2. ✅ Test strategy defined - ready for contract generation
3. ✅ Pagination approach clear - document in API contracts
4. ✅ Caching strategy simple - implement in services layer

**All NEEDS CLARIFICATION items resolved. Proceed to Phase 1: Design & Contracts.**
