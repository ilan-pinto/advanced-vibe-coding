# Data Model: E-Commerce Dashboard

**Feature**: 001-ecommerce-dashboard
**Date**: 2025-10-28
**Purpose**: Define data structures consumed from API gateway

## Overview

This dashboard is **read-only** and consumes data from an existing API gateway. The data models below represent the TypeScript interfaces for data received from the API, not database schemas.

---

## Product

Represents an item available for purchase in the e-commerce catalog.

### TypeScript Interface

```typescript
interface Product {
  id: string;                    // Unique product identifier
  name: string;                  // Product name/title
  price: number;                 // Price in base currency units (e.g., cents)
  currency: string;              // ISO 4217 currency code (e.g., "USD")
  imageUrl?: string;             // Optional product image URL
  description?: string;          // Optional product description
  stock?: number;                // Optional inventory count
  createdAt: string;             // ISO 8601 timestamp
  updatedAt: string;             // ISO 8601 timestamp
}
```

### Validation Rules

- `id`: Required, non-empty string
- `name`: Required, 1-500 characters
- `price`: Required, non-negative number (0 allowed for free items)
- `currency`: Required, 3-letter uppercase ISO code
- `imageUrl`: Optional, valid URL or null
- `stock`: Optional, non-negative integer or null

### Relationships

- Products may appear in Orders (via OrderItem)

### Display Requirements

- **Grid View**: Show name, image (or placeholder), and formatted price
- **Price Format**: Display as currency string (e.g., "$19.99" or "Free" if 0)
- **Image Fallback**: Show placeholder if `imageUrl` is null/invalid
- **Empty State**: "No products available" when array is empty

---

## User

Represents a registered customer or user account.

### TypeScript Interface

```typescript
interface User {
  id: string;                    // Unique user identifier
  email: string;                 // User email address
  name: string;                  // Full name or display name
  avatarUrl?: string;            // Optional avatar image URL
  role?: UserRole;               // Optional user role
  createdAt: string;             // ISO 8601 timestamp (account creation)
  lastLoginAt?: string;          // ISO 8601 timestamp (last login)
}

enum UserRole {
  CUSTOMER = "customer",
  ADMIN = "admin",
  MODERATOR = "moderator"
}
```

### Validation Rules

- `id`: Required, non-empty string
- `email`: Required, valid email format
- `name`: Required, 1-200 characters
- `avatarUrl`: Optional, valid URL or null
- `role`: Optional, one of UserRole enum values

### Relationships

- Users may be customers in Orders (via `customerId`)

### Display Requirements

- **List View**: Show avatar, name, and email
- **Avatar Fallback**: Show initials badge if `avatarUrl` is null/invalid
  - Initials: First letter of first name + first letter of last name
  - Example: "John Doe" → "JD"
- **Empty State**: "No users found" when array is empty

---

## Order

Represents a purchase transaction.

### TypeScript Interface

```typescript
interface Order {
  id: string;                    // Unique order identifier
  customerId: string;            // Reference to User.id
  customerName?: string;         // Denormalized customer name for display
  customerEmail?: string;        // Denormalized customer email
  status: OrderStatus;           // Current order status
  total: number;                 // Total order amount in base currency units
  currency: string;              // ISO 4217 currency code
  items?: OrderItem[];           // Optional order items (may not be included in list view)
  createdAt: string;             // ISO 8601 timestamp (order placed)
  updatedAt: string;             // ISO 8601 timestamp (last status update)
}

enum OrderStatus {
  PENDING = "pending",           // Awaiting processing
  PROCESSING = "processing",     // Being prepared
  SHIPPED = "shipped",           // In transit
  DELIVERED = "delivered",       // Completed successfully
  CANCELLED = "cancelled"        // Not fulfilled
}

interface OrderItem {
  productId: string;             // Reference to Product.id
  productName: string;           // Denormalized product name
  quantity: number;              // Number of units
  priceAtPurchase: number;       // Historical price when ordered
  currency: string;              // ISO 4217 currency code
}
```

### Validation Rules

- `id`: Required, non-empty string
- `customerId`: Required, non-empty string
- `status`: Required, one of OrderStatus enum values
- `total`: Required, non-negative number
- `currency`: Required, 3-letter uppercase ISO code
- `items`: Optional array (may be omitted for list endpoints)
- `quantity`: Positive integer (>= 1)
- `priceAtPurchase`: Non-negative number

### Relationships

- Order → User (via `customerId`)
- OrderItem → Product (via `productId`)

### Display Requirements

- **Table View**: Show order ID, customer name, date, total, and status badge
- **Status Badge**: Visual indicator with distinct colors/icons per status:
  - `PENDING`: Yellow/amber badge with clock icon
  - `PROCESSING`: Blue badge with loading icon
  - `SHIPPED`: Purple badge with truck icon
  - `DELIVERED`: Green badge with checkmark icon
  - `CANCELLED`: Red/gray badge with X icon
- **Date Format**: Display `createdAt` as relative time ("2 hours ago") or absolute ("Jan 15, 2025")
- **Price Format**: Display `total` as currency string (e.g., "$149.99")
- **Empty State**: "No orders to display" when array is empty

---

## Paginated Response

All three API endpoints return paginated data to handle large datasets.

### TypeScript Interface

```typescript
interface PaginatedResponse<T> {
  data: T[];                     // Array of items (Product | User | Order)
  pagination: {
    nextCursor?: string;         // Opaque cursor for next page (null if last page)
    hasMore: boolean;            // Whether more items are available
    total?: number;              // Optional total count across all pages
  }
}
```

### Validation Rules

- `data`: Required array (may be empty)
- `pagination.hasMore`: Required boolean
- `pagination.nextCursor`: Optional string (null/undefined if no more pages)

### Usage

```typescript
// Example: Fetching products
const response: PaginatedResponse<Product> = await api.get('/products?limit=50');
console.log(response.data.length);  // 50 products
console.log(response.pagination.hasMore);  // true
console.log(response.pagination.nextCursor);  // "eyJpZCI6MTIzfQ=="

// Fetch next page
const nextPage = await api.get(`/products?limit=50&cursor=${response.pagination.nextCursor}`);
```

---

## Error Response

Standard error format returned by API gateway when requests fail.

### TypeScript Interface

```typescript
interface ApiError {
  error: {
    code: string;                // Machine-readable error code
    message: string;             // Human-readable error message
    details?: Record<string, any>; // Optional additional error context
  }
  status: number;                // HTTP status code (400, 401, 404, 500, etc.)
}
```

### Common Error Codes

- `NETWORK_ERROR`: Connection failed, timeout
- `UNAUTHORIZED`: Missing or invalid authentication
- `FORBIDDEN`: User lacks permissions
- `NOT_FOUND`: Resource not found
- `RATE_LIMIT`: Too many requests
- `SERVER_ERROR`: Internal server error

### Display Requirements

- **User-Friendly Messages**: Map error codes to user-readable text
  - `NETWORK_ERROR` → "Unable to connect. Check your internet connection."
  - `UNAUTHORIZED` → "Session expired. Please log in again."
  - `SERVER_ERROR` → "Something went wrong. Please try again later."
- **Retry Option**: Show "Try Again" button for transient errors
- **Partial Failure**: If one section fails, show error only for that section (not entire dashboard)

---

## State Transitions

### Order Status Flow

```
PENDING → PROCESSING → SHIPPED → DELIVERED
         ↓
      CANCELLED (terminal state from any prior state)
```

**Business Rules**:
- Orders cannot move backwards (e.g., SHIPPED cannot return to PROCESSING)
- `CANCELLED` and `DELIVERED` are terminal states (no further transitions)
- Status transitions are managed by backend; dashboard displays current state only

---

## Data Freshness

- **Cache TTL**: 5 minutes (stale-while-revalidate)
- **Manual Refresh**: User can force refresh any section
- **Last Updated**: Display timestamp when data was last fetched
- **No Real-Time**: Dashboard does not use websockets/polling (per spec assumptions)

---

## Summary

- **3 main entities**: Product, User, Order
- **Read-only**: No create/update/delete operations
- **Paginated**: All list endpoints return `PaginatedResponse<T>`
- **Currency**: All monetary values stored as base units (cents) with currency code
- **Timestamps**: ISO 8601 strings (convert to Date objects for display)
- **Relationships**: Denormalized for display (customer name in Order, product name in OrderItem)
