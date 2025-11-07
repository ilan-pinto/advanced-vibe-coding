# Quickstart: E-Commerce Dashboard

**Feature**: 001-ecommerce-dashboard
**Date**: 2025-10-28
**Purpose**: Get the dashboard running locally in under 10 minutes

## Prerequisites

- **Node.js**: 20.x LTS ([Download](https://nodejs.org/))
- **Podman**: 4.x+ ([Install guide](https://podman.io/getting-started/installation))
- **API Gateway**: Access to running API gateway (see "API Configuration" below)

---

## Quick Start (Development)

### 1. Clone and Install

```bash
cd frontend
npm install  # CRITICAL: Use npm install, never npm ci (per constitution)
```

### 2. Configure API Gateway

Create `.env` file in `frontend/` directory:

```bash
VITE_API_BASE_URL=https://api.example.com/v1
VITE_API_TIMEOUT=2000
```

Replace `https://api.example.com/v1` with your actual API gateway URL.

### 3. Run Development Server

```bash
npm run dev
```

Dashboard will be available at: **http://localhost:3001**

> **Note**: Port 3001 is required per constitution (port 3000 is prohibited).

### 4. Verify Setup

Open browser to `http://localhost:3001` and verify:

- ✅ Dashboard loads within 3 seconds
- ✅ Products section displays grid view
- ✅ Users section displays list view
- ✅ Orders section displays table view
- ✅ No CORS errors in browser console

---

## Production Deployment (Podman)

### 1. Build Container Image

```bash
cd frontend
podman build -t ecommerce-dashboard:latest -f Containerfile .
```

### 2. Run with podman-compose

Create `compose.yaml` in project root:

```yaml
version: '3.8'

services:
  dashboard:
    image: ecommerce-dashboard:latest
    container_name: ecommerce-dashboard
    ports:
      - "3001:3001"  # Port 3001 required (not 3000)
    environment:
      - VITE_API_BASE_URL=https://api.example.com/v1
      - VITE_API_TIMEOUT=2000
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
```

### 3. Start Services

```bash
podman-compose up -d
```

### 4. Verify Deployment

```bash
podman-compose ps  # Should show "Up" status
curl http://localhost:3001  # Should return HTML
```

Dashboard available at: **http://localhost:3001**

---

## API Configuration

### Required API Endpoints

The dashboard expects these endpoints to be available:

- `GET /products` - List products with pagination
- `GET /users` - List users with pagination
- `GET /orders` - List orders with pagination

### CORS Configuration

API Gateway **must** enable CORS with explicit origin whitelisting:

```javascript
// Example: Express.js CORS config
app.use(cors({
  origin: 'http://localhost:3001',  // Development
  // origin: 'https://dashboard.example.com',  // Production
  credentials: true,
  methods: ['GET', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

**Critical**: Do NOT use wildcard `*` in production (constitution violation).

### Authentication

Authentication is handled by the API gateway. Dashboard expects:

- **JWT token** in `Authorization: Bearer <token>` header
- Token management is out of scope (handled by separate auth system)

---

## Development Scripts

```bash
# Development server (port 3001)
npm run dev

# Run unit tests
npm test

# Run tests with coverage
npm run test:coverage

# Run E2E tests (requires dev server running)
npm run test:e2e

# Build for production
npm run build

# Preview production build locally
npm run preview

# Lint code
npm run lint

# Format code
npm run format
```

---

## Project Structure

```
frontend/
├── src/
│   ├── components/       # React components
│   ├── services/         # API client logic
│   ├── hooks/            # Custom React hooks
│   ├── utils/            # Helper functions
│   └── styles/           # CSS files
├── tests/
│   ├── integration/      # API integration tests
│   ├── components/       # Component tests
│   └── e2e/              # End-to-end tests
├── public/               # Static assets
├── Containerfile         # Podman container definition
├── compose.yaml          # Podman compose configuration
├── package.json          # Dependencies
├── vite.config.js        # Vite build config
└── .env                  # Environment variables (create this)
```

---

## Testing

### Unit & Component Tests (Vitest)

```bash
npm test

# Watch mode
npm test -- --watch

# Coverage report
npm run test:coverage
# Open coverage/index.html in browser
```

### E2E Tests (Playwright)

```bash
# Install Playwright browsers (first time only)
npx playwright install

# Run E2E tests
npm run test:e2e

# Run with UI
npm run test:e2e -- --ui
```

---

## Troubleshooting

### Dashboard Won't Load

**Symptom**: Blank page or "Cannot GET /" error

**Solution**:
```bash
# Verify dev server is running on port 3001
lsof -i :3001

# Check for port conflicts
kill -9 $(lsof -t -i :3001)  # Kill process on port 3001
npm run dev  # Restart dev server
```

### CORS Errors

**Symptom**: Browser console shows "CORS policy blocked" errors

**Solutions**:
1. Verify API gateway CORS configuration allows origin `http://localhost:3001`
2. Check API gateway is running and accessible
3. Verify `.env` has correct `VITE_API_BASE_URL`
4. Test API endpoint directly: `curl http://api.example.com/v1/products`

### API Timeouts

**Symptom**: "Request timeout" or "Network error" messages

**Solutions**:
1. Increase timeout in `.env`: `VITE_API_TIMEOUT=5000`
2. Check API gateway logs for slow queries
3. Verify network connectivity: `ping api.example.com`
4. Test API directly: `curl -w "@curl-format.txt" http://api.example.com/v1/products`

### Podman Build Fails

**Symptom**: `podman build` command fails

**Solutions**:
```bash
# Verify Podman is installed and running
podman --version

# Check for image name conflicts
podman images | grep ecommerce-dashboard
podman rmi ecommerce-dashboard:latest  # Remove old image

# Rebuild with verbose output
podman build --no-cache -t ecommerce-dashboard:latest -f Containerfile .
```

### npm install Errors

**Symptom**: Dependency installation fails

**Solutions**:
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and lock file
rm -rf node_modules package-lock.json

# Reinstall (remember: npm install, not npm ci!)
npm install
```

---

## Performance Verification

After deployment, verify performance requirements:

```bash
# Load time test
curl -w "@curl-format.txt" -o /dev/null -s http://localhost:3001

# Expected: Total time < 3 seconds
```

### Browser DevTools Checklist

1. **Network Tab**: Verify API calls < 2 seconds
2. **Console Tab**: No CORS errors, no JavaScript errors
3. **Performance Tab**: LCP < 2.5s, FID < 100ms, CLS < 0.1

---

## Next Steps

1. ✅ Dashboard running locally
2. ✅ API connectivity verified
3. ✅ CORS working correctly
4. 📋 Ready for implementation: `/speckit.tasks`

---

## Configuration Reference

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `VITE_API_BASE_URL` | Yes | - | API gateway base URL |
| `VITE_API_TIMEOUT` | No | 2000 | Request timeout (ms) |
| `VITE_PORT` | No | 3001 | Dev server port |
| `VITE_CACHE_TTL` | No | 300000 | Cache TTL (ms, 5 min) |

### Containerfile

```dockerfile
# Multi-stage build for minimal image size (per constitution)
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install  # CRITICAL: npm install, not npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runtime

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm install --only=production  # CRITICAL: npm install, not npm ci

EXPOSE 3001
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3001', (res) => process.exit(res.statusCode === 200 ? 0 : 1))"

CMD ["npm", "run", "preview", "--", "--port", "3001", "--host", "0.0.0.0"]
```

---

## Support

- **Documentation**: See `spec.md`, `plan.md`, `data-model.md`, `contracts/api-gateway.yaml`
- **Issues**: Check browser console and API gateway logs first
- **Performance**: Use Chrome DevTools Performance tab to profile slow loads
