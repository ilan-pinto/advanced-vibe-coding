# Terminal 1 - Product Service 
claude --dangerously-skip-permissions "
as part of the microservices demo create a folder ./product-service 
Create a Node.js Express microservice for products on port 8001. Include endpoints for listing products and getting product details. Use in-memory storage with 10 sample products including name, price, description, and stock quantity. Add CORS support. 

- Node.js Express service running on PORT environment variable (8001)
- Multi-stage Dockerfile with 'production' target
- IMPORTANT: Use 'npm install' (NOT npm ci) in Dockerfile
- Endpoints:
  - GET /health - Health check returning { status: 'healthy' }
  - GET /products - List all products
  - GET /products/:id - Get single product
  - POST /products - Create product
- In-memory storage with 10 sample products (electronics, books, clothing)
- Each product: { id, name, price, description, category, stock, imageUrl }
- CORS enabled for all origins

" 
# Terminal 2 - User Service 
claude --dangerously-skip-permissions "
as part of the microservices demo create a folder ./user-service  
Create a Python FastAPI microservice for users on port 8002. Include endpoints for listing users and getting user profiles. Use in-memory storage with 5 sample users including name, email, and registration date. Add CORS support.
- Python FastAPI service running on PORT environment variable (8002)
- Multi-stage Dockerfile with 'production' target
- IMPORTANT: In Dockerfile, copy Python packages to /home/appuser/.local instead of /root/.local so the non-root user can access them
- Endpoints:
  - GET /health - Health check
  - GET /users - List all users
  - GET /users/{id} - Get user details
  - POST /users - Create user
- In-memory storage with 5 sample users
- Each user: { id, name, email, avatar, registrationDate, role }
- CORS middleware configured

"
 # Terminal 3 - Order Service 
 claude --dangerously-skip-permissions "
 as part of the microservices demo create a folder ./order-service 
 Create a Node.js Express microservice for orders on port 8003. Include endpoints for creating orders and listing orders by user. Orders should reference userId and productIds. Include order status (pending/shipped/delivered) and timestamps. Add CORS support. 
 - Node.js Express service on PORT environment variable (8003)
- Multi-stage Dockerfile with 'production' target
- IMPORTANT: Use 'npm install' (NOT npm ci) in Dockerfile
- Use PRODUCT_SERVICE_URL and USER_SERVICE_URL from environment
- Endpoints:
  - GET /health - Health check
  - GET /orders - List all orders
  - GET /orders/user/:userId - Get user's orders
  - POST /orders - Create order (validate user and products exist)
- Each order: { id, userId, products: [{productId, quantity}], status, total, createdAt }
- Status enum: ['pending', 'processing', 'shipped', 'delivered']
- Make HTTP calls to product and user services for validation"
" 

 # Terminal 4 -  API Gateway
 claude --dangerously-skip-permissions "
 as part of the microservices demo create a folder ./api-gateway
 Create a Node.js Express API gateway on port 8000 that routes requests to microservices: /api/products -> localhost:8001, /api/users -> localhost:8002, /api/orders -> localhost:8003. Use http-proxy-middleware for routing. Add CORS support.
 - Node.js Express gateway on PORT environment variable (8000)
- Multi-stage Dockerfile with 'production' target
- IMPORTANT: Use 'npm install' (NOT npm ci) in Dockerfile
- Use service URLs from environment variables
- Route configuration:
  - /api/products/* -> PRODUCT_SERVICE_URL (rewrite /api/products to /products, NOT to /)
  - /api/users/* -> USER_SERVICE_URL (rewrite /api/users to /users, NOT to /)
  - /api/orders/* -> ORDER_SERVICE_URL (rewrite /api/orders to /orders, NOT to /)
  - /health -> aggregate health check of all services
- Use http-proxy-middleware for routing with pathRewrite to keep the service path
- CORS enabled for all origins
- Request logging middleware " 
# Terminal 5 - Frontend UI 

claude --dangerously-skip-permissions "
as part of the microservices demo create a folder ./frontend
Create a React dashboard that displays data from an API gateway at localhost:8000. Show three sections: Products (grid view with prices), Users (list with avatars), and Orders (table with status badges). Use Tailwind CSS for styling. Auto-refresh data every 5 seconds. 

- React application with Tailwind CSS running on port 3001
- Multi-stage Dockerfile with nginx for production target (nginx should listen on port 8080, not 80, since non-root users can't bind to ports < 1024)
- IMPORTANT: Use 'npm install' (NOT npm ci) in Dockerfile
- Use REACT_APP_API_GATEWAY_URL from environment
- In docker-compose.yml, map port 3001:8080 and update healthcheck to use port 8080
- Frontend should fetch data from these exact URLs:
  - http://localhost:8000/api/products
  - http://localhost:8000/api/users
  - http://localhost:8000/api/orders
- Dashboard with three sections:
  - Products Grid: Card layout with images, prices, stock badges
  - Users List: Table with avatars, email, registration date
  - Orders Dashboard: Table with order details, status badges with colors
  - Summary cards showing total counts
- Auto-refresh data every 5 seconds
- Loading states and error handling
- Modern, clean design with shadows and hover effects"

# Terminal 6
claude --dangerously-skip-permissions "Create a complete containerized of the below  services which are under the below folders:

## Project Structure
Create a root directory with 5 service subdirectories:
- ./product-service
- ./user-service  
- ./order-service
- ./api-gateway
- ./frontend
ß

## Docker Configuration

Each service needs a multi-stage Dockerfile:
- Build stage: Install dependencies using `npm install` (not npm ci) and build (if needed)
- Production stage: Minimal runtime image with only necessary files
- Health check commands in Dockerfile
- Non-root user for running the service

## Root Files

### docker-compose.yml
Use the exact structure provided:
- All services with 'production' build target
- microservices-network bridge network
- Proper service dependencies
- Environment variables as specified
- Restart policies set to unless-stopped

### Makefile
Commands:
- build: podman-compose build
- up: podman-compose up -d
- down: podman-compose down
- logs: podman-compose logs -f
- restart: make down && make up
- status: podman-compose ps

### README.md
Include:
- ASCII architecture diagram
- Service descriptions
- Startup instructions
- API endpoint documentation
- Testing commands with curl examples

## Testing
Before completing:
1. Ensure all services build successfully
2. Verify inter-service communication works
3. Confirm frontend displays data from all services
4. Test health endpoints respond correctly

Create all files with proper error handling, logging, and production best practices."