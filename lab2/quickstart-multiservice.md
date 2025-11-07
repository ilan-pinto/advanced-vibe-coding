# Quickstart: E-Commerce Dashboard (Microservices)

**Feature**: 001-ecommerce-dashboard
**Date**: 2025-10-28
**Purpose**: Get all 5 services running for parallel team development

## Architecture

```
Frontend (React) :3001 → API Gateway (Express) :8000 → Product Service (FastAPI) :8001
                                                     → User Service (Gin) :8002
                                                     → Order Service (Actix) :8003
                                                                           ↓
                                                              PostgreSQL :5432
```

## Prerequisites by Team

| Team | Language | Prerequisites |
|------|----------|---------------|
| Frontend | TypeScript | Node.js 20.x LTS, npm |
| Product | Python | Python 3.11+, pip |
| User | Go | Go 1.21+, go mod |
| Order | Rust | Rust 1.75+, cargo |
| Platform (Gateway) | TypeScript | Node.js 20.x LTS, npm |
| **All Teams** | - | **Podman 4.x+**, PostgreSQL 15+ |

---

## Option 1: Full Stack (All Services via Podman Compose)

**Best for**: Integration testing, demos, full-stack developers

### 1. Start Database

```bash
podman run -d \
  --name ecommerce-db \
  -e POSTGRES_PASSWORD=devpassword \
  -e POSTGRES_DB=ecommerce \
  -p 5432:5432 \
  postgres:15-alpine
```

### 2. Create Database Schemas

```bash
podman exec -it ecommerce-db psql -U postgres -d ecommerce -c "
CREATE SCHEMA products;
CREATE SCHEMA users;
CREATE SCHEMA orders;
"
```

### 3. Build All Services

```bash
# Product Service
cd product-service
podman build -t product-service:latest -f Containerfile .

# User Service
cd ../user-service
podman build -t user-service:latest -f Containerfile .

# Order Service
cd ../order-service
podman build -t order-service:latest -f Containerfile .

# API Gateway
cd ../api-gateway
podman build -t api-gateway:latest -f Containerfile .

# Frontend
cd ../frontend
podman build -t frontend:latest -f Containerfile .
```

### 4. Start All Services with Compose

```bash
podman-compose up -d
```

### 5. Verify

```bash
curl http://localhost:3001  # Frontend
curl http://localhost:8000/health  # API Gateway
curl http://localhost:8001/health  # Product Service
curl http://localhost:8002/health  # User Service
curl http://localhost:8003/health  # Order Service
```

Dashboard: **http://localhost:3001**

---

## Option 2: Individual Service Development (Recommended for Teams)

**Best for**: Parallel development, each team works independently

### Prerequisites: Database Setup

All teams need PostgreSQL running:

```bash
podman run -d \
  --name ecommerce-db \
  -e POSTGRES_PASSWORD=devpassword \
  -e POSTGRES_DB=ecommerce \
  -p 5432:5432 \
  postgres:15-alpine

# Create schemas
podman exec -it ecommerce-db psql -U postgres -d ecommerce -c "
CREATE SCHEMA products;
CREATE SCHEMA users;
CREATE SCHEMA orders;
"
```

---

### Frontend Team (TypeScript/React)

**Can start immediately** - Use mocked API during development

```bash
cd frontend

# Install dependencies
npm install  # CRITICAL: Never use npm ci

# Create .env
cat > .env << EOF
VITE_API_BASE_URL=http://localhost:8000
VITE_API_TIMEOUT=2000
EOF

# Run dev server
npm run dev
```

Dashboard at: **http://localhost:3001**

**Mock API** (until gateway is ready):
```typescript
// frontend/src/services/__mocks__/api.ts
export const mockProducts = [
  { id: "1", name: "Product 1", price: 1999, currency: "USD" }
];
```

---

### Product Team (Python/FastAPI)

**Can start immediately** - Independent service

```bash
cd product-service

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Create .env
cat > .env << EOF
DATABASE_URL=postgresql://postgres:devpassword@localhost:5432/ecommerce
DB_SCHEMA=products
PORT=8001
EOF

# Run migrations
alembic upgrade head

# Run service
uvicorn app.main:app --host 0.0.0.0 --port 8001 --reload
```

Service at: **http://localhost:8001**

**Test endpoint**:
```bash
curl http://localhost:8001/products?limit=10
```

---

### User Team (Go/Gin)

**Can start immediately** - Independent service

```bash
cd user-service

# Install dependencies
go mod download

# Create .env
cat > .env << EOF
DATABASE_URL=postgres://postgres:devpassword@localhost:5432/ecommerce?search_path=users
PORT=8002
EOF

# Run migrations
migrate -path migrations -database "${DATABASE_URL}" up

# Run service
go run cmd/server/main.go
```

Service at: **http://localhost:8002**

**Test endpoint**:
```bash
curl http://localhost:8002/users?limit=10
```

---

### Order Team (Rust/Actix)

**Can start immediately** - Independent service

```bash
cd order-service

# Create .env
cat > .env << EOF
DATABASE_URL=postgres://postgres:devpassword@localhost:5432/ecommerce
DB_SCHEMA=orders
PORT=8003
EOF

# Run migrations
sqlx migrate run

# Build and run
cargo run
```

Service at: **http://localhost:8003**

**Test endpoint**:
```bash
curl http://localhost:8003/orders?limit=10
```

---

### Platform Team (API Gateway)

**Starts after backend services are ready** - Aggregates all services

```bash
cd api-gateway

# Install dependencies
npm install  # CRITICAL: Never use npm ci

# Create .env
cat > .env << EOF
PORT=8000
PRODUCT_SERVICE_URL=http://localhost:8001
USER_SERVICE_URL=http://localhost:8002
ORDER_SERVICE_URL=http://localhost:8003
CORS_ORIGIN=http://localhost:3001
JWT_SECRET=your-secret-key-here
EOF

# Run gateway
npm run dev
```

Gateway at: **http://localhost:8000**

**Test aggregation**:
```bash
curl http://localhost:8000/products
curl http://localhost:8000/users
curl http://localhost:8000/orders
```

---

## Development Workflow

### Phase 1: Contract Agreement (Day 1-2)
All teams meet to finalize API contracts in `specs/001-ecommerce-dashboard/contracts/`

### Phase 2: Parallel Development (Week 1-2)
Each team works independently in their service directory:
- Frontend: Build UI with mocks
- Backend services: Implement endpoints + tests
- Gateway: Implement routing once backends are ready

### Phase 3: Integration (Week 2-3)
- Frontend switches from mocks to real gateway
- E2E testing across all services
- Performance tuning

---

## Testing by Team

### Frontend
```bash
cd frontend
npm test  # Vitest unit tests
npm run test:e2e  # Playwright E2E tests
```

### Product Service
```bash
cd product-service
pytest  # All tests
pytest tests/test_products.py  # Specific test
```

### User Service
```bash
cd user-service
go test ./...  # All tests
go test -v ./internal/handlers  # Specific package
```

### Order Service
```bash
cd order-service
cargo test  # All tests
cargo test --test integration_tests  # Integration only
```

### API Gateway
```bash
cd api-gateway
npm test  # Vitest + Supertest
```

---

## Containerization (Constitution Compliance)

Each service has its own `Containerfile` (Podman-compatible):

### Example: Product Service Containerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8001
HEALTHCHECK --interval=30s --timeout=10s \
  CMD curl -f http://localhost:8001/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8001"]
```

**Build**:
```bash
podman build -t product-service:latest -f Containerfile .
```

**Run**:
```bash
podman run -d -p 8001:8001 \
  -e DATABASE_URL=postgresql://... \
  product-service:latest
```

---

## Compose Configuration (Full Stack)

**compose.yaml** (root directory):

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: ecommerce
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 10s

  product-service:
    image: product-service:latest
    ports:
      - "8001:8001"
    environment:
      DATABASE_URL: postgresql://postgres:devpassword@postgres:5432/ecommerce
      DB_SCHEMA: products
    depends_on:
      - postgres
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  user-service:
    image: user-service:latest
    ports:
      - "8002:8002"
    environment:
      DATABASE_URL: postgres://postgres:devpassword@postgres:5432/ecommerce?search_path=users
    depends_on:
      - postgres
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  order-service:
    image: order-service:latest
    ports:
      - "8003:8003"
    environment:
      DATABASE_URL: postgres://postgres:devpassword@postgres:5432/ecommerce
      DB_SCHEMA: orders
    depends_on:
      - postgres
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  api-gateway:
    image: api-gateway:latest
    ports:
      - "8000:8000"
    environment:
      PRODUCT_SERVICE_URL: http://product-service:8001
      USER_SERVICE_URL: http://user-service:8002
      ORDER_SERVICE_URL: http://order-service:8003
      CORS_ORIGIN: http://localhost:3001
    depends_on:
      - product-service
      - user-service
      - order-service
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  frontend:
    image: frontend:latest
    ports:
      - "3001:3001"  # Port 3001 required (not 3000)
    environment:
      VITE_API_BASE_URL: http://localhost:8000
    depends_on:
      - api-gateway
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

volumes:
  pgdata:
```

---

## Troubleshooting

### Service Won't Start

**Check logs**:
```bash
podman logs <service-name>
podman logs -f product-service  # Follow logs
```

### Database Connection Fails

**Verify PostgreSQL**:
```bash
podman exec -it ecommerce-db psql -U postgres -d ecommerce -c "\dn"
# Should show: products, users, orders schemas
```

### Port Conflicts

**Check ports**:
```bash
lsof -i :3001  # Frontend
lsof -i :8000  # Gateway
lsof -i :8001  # Product
lsof -i :8002  # User
lsof -i :8003  # Order
```

### Service Can't Reach Another Service

**In Podman compose**: Use service names as hostnames (`http://product-service:8001`)
**On host**: Use `localhost` (`http://localhost:8001`)

---

## Performance Verification

```bash
# Load time (should be < 3 seconds)
time curl -s http://localhost:3001 > /dev/null

# API response time (should be < 500ms p95)
for i in {1..100}; do
  curl -w "%{time_total}\n" -o /dev/null -s http://localhost:8000/products
done | sort -n | tail -5
```

---

## Next Steps

1. ✅ All services running
2. ✅ Database schemas created
3. ✅ Services can communicate
4. 📋 Ready for implementation: `/speckit.tasks`
