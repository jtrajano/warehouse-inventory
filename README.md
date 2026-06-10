# Technical Design Document
## Warehouse & Logistics Inventory Management System

**Version:** 1.0  
**Author:** Jeff  
**Created:** June 2026  
**Status:** Active Development

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Goals & Success Criteria](#2-goals--success-criteria)
3. [Stack Decisions](#3-stack-decisions)
4. [System Architecture](#4-system-architecture)
5. [MVP Scope](#5-mvp-scope)
6. [Data Model](#6-data-model)
7. [API Design Principles](#7-api-design-principles)
8. [Frontend Architecture](#8-frontend-architecture)
9. [Backend Architecture](#9-backend-architecture)
10. [Security](#10-security)
11. [Development Approach](#11-development-approach)
12. [Sprint Plan Summary](#12-sprint-plan-summary)
13. [Out of Scope](#13-out-of-scope)
14. [Open Questions](#14-open-questions)

---

## 1. Project Overview

The Warehouse & Logistics Inventory Management System is a full-stack web application designed to give warehouse operators real-time visibility into stock levels, inbound and outbound order tracking, and supplier coordination — all from a single interface.

The system targets small-to-medium warehouse operations that currently manage inventory through spreadsheets or disconnected tools and need a structured, auditable alternative.

This project serves two purposes:

- **Portfolio showcase** — demonstrating production-grade full-stack development using modern .NET and React patterns
- **Real-world use** — designed with actual warehouse workflows in mind, not as a toy demo

---

## 2. Goals & Success Criteria

### Primary Goals

| # | Goal |
|---|------|
| G1 | Provide real-time stock visibility across multiple warehouse locations |
| G2 | Track every stock movement with an immutable audit trail |
| G3 | Manage inbound purchase orders and outbound dispatch orders end-to-end |
| G4 | Alert operators when stock falls below defined thresholds |
| G5 | Support role-based access so different team members see what they need |

### Success Criteria (MVP)

The system is considered MVP-complete when a warehouse operator can:

1. Log in and land on a dashboard showing current stock health
2. Browse products, search by SKU or name, and view stock by location
3. Create and receive a purchase order — stock increases automatically
4. Create and dispatch an outbound order — stock decreases automatically
5. Manually adjust stock with a required reason code
6. See low stock alerts on the dashboard and alerts page
7. Export a current stock level report to CSV

---

## 3. Stack Decisions

### 3.1 Frontend — React 18 + TypeScript

**Decision:** React 18 with TypeScript using Vite as the build tool.

**Rationale:**
- React is the dominant frontend framework in enterprise and startup contexts — maximum portfolio signal
- TypeScript eliminates a large class of runtime errors and enforces contracts between frontend and backend
- Vite provides significantly faster dev server startup and HMR compared to Create React App

**Alternatives considered:**

| Alternative | Reason Not Chosen |
|---|---|
| Next.js | SSR adds complexity not needed for a SPA dashboard; no SEO requirement |
| Vue 3 | Smaller ecosystem for enterprise tooling; less portfolio visibility |
| Angular | Heavier framework overhead; overkill for this scope |

### 3.2 UI Layer — shadcn/ui + Tailwind CSS

**Decision:** shadcn/ui as the component library with Tailwind CSS for styling.

**Rationale:**
- shadcn/ui gives full ownership of component code — no black-box library overrides
- Tailwind eliminates context-switching between CSS files and JSX
- The combination produces polished, accessible UIs with minimal custom CSS
- shadcn/ui uses Radix UI primitives which are fully accessible out of the box

**Alternatives considered:**

| Alternative | Reason Not Chosen |
|---|---|
| MUI (Material UI) | Heavy bundle, opinionated Material Design look |
| Chakra UI | Less flexible theming; less portfolio differentiation |
| Ant Design | Enterprise-heavy aesthetic; large bundle size |

### 3.3 Server State — TanStack Query

**Decision:** TanStack Query (React Query) for all server state management.

**Rationale:**
- Handles caching, background refetching, loading/error states, and optimistic updates without boilerplate
- Eliminates the need for Redux or Zustand for server data
- The mock → real API swap strategy relies on the hook interface staying identical — TanStack Query makes this seamless

### 3.4 Forms — React Hook Form + Zod

**Decision:** React Hook Form for form state with Zod for schema validation.

**Rationale:**
- React Hook Form is uncontrolled by default — minimal re-renders on keypress
- Zod provides type-safe schema validation that can be shared between frontend and backend (if BFF pattern is adopted later)
- The `@hookform/resolvers/zod` adapter bridges the two with no boilerplate

### 3.5 Backend — .NET 8 Web API

**Decision:** .NET 8 minimal API / controller-based Web API.

**Rationale:**
- .NET 8 is the current LTS release with strong performance benchmarks
- C# is strongly typed end-to-end — matches the TypeScript philosophy on the frontend
- ASP.NET Core has first-class support for JWT auth, EF Core, and OpenAPI
- Strong portfolio signal for enterprise and finance contexts common in the Philippines and global remote market

**Alternatives considered:**

| Alternative | Reason Not Chosen |
|---|---|
| Node.js / Express | Less type safety; less differentiation given React is already JS |
| Python / FastAPI | Lower enterprise signal; slower execution for data-heavy operations |
| Go | Excellent performance but smaller community for RAD; less ORM maturity |

### 3.6 ORM — Entity Framework Core 8

**Decision:** EF Core 8 with code-first migrations.

**Rationale:**
- Code-first keeps the domain model as the source of truth
- Migrations are version-controlled alongside application code
- LINQ queries are strongly typed — no raw SQL string errors

### 3.7 Database — PostgreSQL

**Decision:** PostgreSQL as the primary data store.

**Rationale:**
- Open source with no licensing cost
- Excellent JSON support for future flexible attributes (product metadata)
- Strong support in cloud platforms (Railway, Supabase, Render, AWS RDS)
- Works cleanly with EF Core via `Npgsql.EntityFrameworkCore.PostgreSQL`

**Alternatives considered:**

| Alternative | Reason Not Chosen |
|---|---|
| SQL Server | Licensing cost in production; heavier footprint |
| SQLite | Not suitable for multi-user concurrent writes |
| MySQL | PostgreSQL has stronger JSON and indexing features |

### 3.8 Authentication — JWT + Refresh Tokens

**Decision:** Stateless JWT access tokens with sliding refresh tokens stored in HTTP-only cookies.

**Rationale:**
- Stateless access tokens reduce database load on every authenticated request
- HTTP-only cookie for refresh token prevents XSS access to long-lived credentials
- Standard approach well understood by hiring managers reviewing the portfolio

---

## 4. System Architecture

```
┌─────────────────────────────────────────────────────┐
│                     Browser                          │
│                                                      │
│   React 18 + TypeScript (Vite)                      │
│   TanStack Query · React Router · shadcn/ui          │
└───────────────────┬─────────────────────────────────┘
                    │ HTTPS / REST JSON
                    ▼
┌─────────────────────────────────────────────────────┐
│               .NET 8 Web API                         │
│                                                      │
│   Controllers → Services → Repositories             │
│   FluentValidation · AutoMapper · JWT Auth           │
│   Swagger / Scalar (API Docs)                        │
└───────────────────┬─────────────────────────────────┘
                    │ EF Core 8
                    ▼
┌─────────────────────────────────────────────────────┐
│              PostgreSQL Database                     │
└─────────────────────────────────────────────────────┘
```

### Project Structure

```
/warehouse-inventory
├── /frontend                     # React + Vite app
│   └── /src
│       ├── /components           # Shared UI components
│       ├── /pages                # Route-level page components
│       ├── /hooks                # TanStack Query hooks (mock + real)
│       ├── /mocks                # Mock data and mock hook implementations
│       ├── /types                # TypeScript domain types
│       ├── /context              # React context (auth, theme)
│       └── /lib                  # Axios instance, utilities
│
├── /backend                      # .NET solution
│   ├── /InventoryAPI             # Web API project (controllers, middleware)
│   ├── /InventoryAPI.Core        # Domain entities, interfaces, DTOs
│   ├── /InventoryAPI.Infrastructure  # EF Core, repositories, services
│   └── /InventoryAPI.Tests       # xUnit tests
│
├── /docs                         # PRD, TDD, ERD
├── docker-compose.yml
└── README.md
```

---

## 5. MVP Scope

The MVP is intentionally scoped to deliver a complete end-to-end flow — not a feature-complete system. Every feature in the MVP must work fully before any stretch feature is started.

### In MVP

| Module | Features |
|---|---|
| **Auth** | Login, logout, JWT refresh, role-based access (Admin, Manager, Operator, Viewer) |
| **Products** | CRUD, SKU/barcode, category, supplier, unit of measure, min stock threshold |
| **Categories** | CRUD |
| **Suppliers** | CRUD |
| **Locations** | Warehouse → Zone → Bin hierarchy, CRUD |
| **Purchase Orders** | Create, submit, receive (full/partial), status progression |
| **Outbound Orders** | Create, dispatch, status progression |
| **Transactions** | Immutable log of all stock movements, manual adjustment with reason code |
| **Stock Levels** | Current qty per product per location, calculated from transaction history |
| **Dashboard** | KPI cards, stock movement chart, low stock list, warehouse capacity |
| **Alerts** | Low stock alerts auto-generated on stock mutation, overdue PO alerts |
| **Reports** | Current stock level report (CSV export) |

### Post-MVP (Backlog)

- Stock transfer between locations/warehouses
- Barcode scanner integration
- Email notifications
- Inventory valuation report (FIFO / average cost)
- Supplier performance report
- PDF report export
- Real-time updates (WebSockets / SignalR)
- Mobile-responsive optimizations
- Docker deployment to cloud

---

## 6. Data Model

### Core Entities

```
User
  id, email, passwordHash, firstName, lastName, role, isActive, createdAt

Product
  id, sku, barcode, name, description, categoryId, supplierId,
  unitOfMeasure, weight, dimensionsL/W/H, minStockThreshold,
  isArchived, createdAt, updatedAt

Category
  id, name, description

Supplier
  id, name, contactPerson, email, phone, address,
  leadTimeDays, paymentTerms, isActive

Warehouse
  id, name, address, isActive

Zone
  id, warehouseId, name, description

Location (Bin)
  id, zoneId, aisle, bay, level, capacity, isActive
  — display format: WH-A / Zone-2 / A-04-B

StockLevel
  id, productId, locationId, quantity, lastUpdated
  — materialized from InventoryTransaction history

PurchaseOrder
  id, supplierId, status, expectedDeliveryDate,
  notes, createdById, createdAt, updatedAt
  status: Draft | Submitted | PartiallyReceived | Complete | Cancelled

PurchaseOrderLine
  id, purchaseOrderId, productId, orderedQty, receivedQty, unitCost

OutboundOrder
  id, referenceNumber, status, dispatchDate,
  notes, createdById, createdAt
  status: Pending | Picked | Dispatched | Cancelled

OutboundOrderLine
  id, outboundOrderId, productId, locationId, requestedQty, dispatchedQty

InventoryTransaction  (immutable — never updated or deleted)
  id, productId, locationId, transactionType,
  quantity, referenceId, referenceType, notes,
  performedById, performedAt
  type: Receive | Dispatch | Transfer | Adjust | Return | Damage

Alert
  id, alertType, productId, purchaseOrderId,
  message, isRead, createdAt
  type: LowStock | OverduePO
```

### Key Relationships

- `Product` belongs to one `Category` and one `Supplier`
- `StockLevel` is the current snapshot of `Product` quantity at a `Location`
- Every mutation to `StockLevel` creates an `InventoryTransaction` entry
- `PurchaseOrder` receiving creates `InventoryTransaction` of type `Receive`
- `OutboundOrder` dispatch creates `InventoryTransaction` of type `Dispatch`
- `Alert` is auto-created by the `InventoryService` when stock drops below `minStockThreshold`

---

## 7. API Design Principles

- RESTful resource-based routing: `GET /api/products`, `POST /api/products`, `GET /api/products/{id}`
- Consistent response envelope:
```json
{
  "data": {},
  "meta": { "page": 1, "pageSize": 25, "total": 120 },
  "errors": []
}
```
- Pagination via query params: `?page=1&pageSize=25`
- Filtering via query params: `?categoryId=3&status=LowStock&search=widget`
- Sorting via query params: `?sortBy=name&sortDir=asc`
- All timestamps in ISO 8601 UTC
- HTTP status codes used correctly (200, 201, 204, 400, 401, 403, 404, 422, 500)
- Validation errors return 422 with field-level error detail
- API versioned via URL prefix: `/api/v1/`

### Key Endpoints

```
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/auth/logout

GET    /api/v1/products
POST   /api/v1/products
GET    /api/v1/products/{id}
PUT    /api/v1/products/{id}
DELETE /api/v1/products/{id}

GET    /api/v1/purchase-orders
POST   /api/v1/purchase-orders
GET    /api/v1/purchase-orders/{id}
POST   /api/v1/purchase-orders/{id}/receive

GET    /api/v1/outbound-orders
POST   /api/v1/outbound-orders
POST   /api/v1/outbound-orders/{id}/dispatch

GET    /api/v1/transactions
POST   /api/v1/transactions/adjust

GET    /api/v1/alerts
PATCH  /api/v1/alerts/{id}/dismiss

GET    /api/v1/dashboard/stats

GET    /api/v1/reports/stock-levels
```

---

## 8. Frontend Architecture

### State Management Strategy

| State Type | Tool |
|---|---|
| Server state (API data) | TanStack Query |
| Form state | React Hook Form |
| Global UI state (auth, theme) | React Context |
| Local component state | useState / useReducer |

Redux is intentionally excluded. The combination of TanStack Query + Context covers all state needs without the boilerplate.

### Mock-First Development

All TanStack Query hooks are written once with a mock implementation. When the backend is ready, only the internals of each hook change — the component consuming `useProducts()` never changes.

```typescript
// Phase 2 (mock)
export function useProducts(filters?: ProductFilters) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => Promise.resolve(mockProducts.filter(...))
  })
}

// Phase 4 (real API — same hook signature)
export function useProducts(filters?: ProductFilters) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => api.get('/products', { params: filters }).then(r => r.data)
  })
}
```

### Routing Strategy

React Router v6 with a layout-based route structure. Protected routes check `isAuthenticated` from `AuthContext` and redirect to `/login` if false. Role-based UI guards hide or disable elements based on the current user's role.

---

## 9. Backend Architecture

### Layer Separation

```
Controller     → validates HTTP request, calls service, returns response
Service        → business logic, orchestrates repositories
Repository     → data access via EF Core DbContext
Domain Entity  → plain C# class, no logic
DTO            → request/response shapes, mapped via AutoMapper
```

### Key Services

- `InventoryService` — core stock mutation logic; receives POs, dispatches orders, adjusts stock, creates transactions, triggers alerts
- `AlertService` — checks thresholds after every stock mutation; creates or resolves alerts
- `ReportService` — aggregates data for report endpoints and CSV export
- `AuthService` — JWT generation, refresh token rotation, password hashing (BCrypt)

### Validation

All incoming request DTOs are validated by FluentValidation. Validation errors are caught by a global exception middleware and returned as 422 responses with field-level detail.

### Testing Strategy

- **Unit tests** (`xUnit` + `Moq`): `InventoryService`, `AlertService`, validation rules
- **Integration tests** (`WebApplicationFactory` + `Testcontainers`): full HTTP request → response tests against a real PostgreSQL container

---

## 10. Security

| Concern | Approach |
|---|---|
| Authentication | JWT access token (15 min expiry) + refresh token (7 day, HTTP-only cookie) |
| Password storage | BCrypt with salt (via `BCrypt.Net-Next`) |
| Authorization | Role-based `[Authorize(Roles = "...")]` on controllers |
| Input validation | FluentValidation on all DTOs; EF Core parameterized queries (no raw SQL) |
| CORS | Explicit allow-list of frontend origins |
| HTTPS | Enforced in production; redirected from HTTP |
| Secrets | Never committed — managed via environment variables and `.env` files |

---

## 11. Development Approach

### UI-First, Contract-Driven

The frontend is built first against mock data. The mock hooks define the exact data contract that the backend must fulfill. This means:

1. The full UI is usable and demoable before a single backend line is written
2. The API shape is designed from the consumer's perspective, not the database's
3. Sprint velocity is higher in early sprints — visual progress is immediate

### Definition of Done (per issue)

- [ ] Feature works against mock data (frontend issues) or passes Swagger test (backend issues)
- [ ] TypeScript compiles with no errors (`tsc --noEmit`)
- [ ] No ESLint warnings
- [ ] Loading, error, and empty states handled
- [ ] Code reviewed (self-review for solo dev: read your diff before merging)
- [ ] GitHub issue closed with brief completion note

---

## 12. Sprint Plan Summary

| Sprint | Week | Focus | Exit Criteria |
|---|---|---|---|
| Sprint 1 | Week 1 | Setup, UI shell, routing, mock data layer, Dashboard, Products | Full app navigable with mock data |
| Sprint 2 | Week 2 | Locations, POs, Outbound Orders, Transactions, Alerts, Suppliers UI | All UI modules complete on mock data |
| Sprint 3 | Week 3 | .NET backend — domain, EF Core, all API endpoints | All endpoints live, Swagger docs complete |
| Sprint 4 | Week 4 | Swap mocks for real API, auth flow, end-to-end integration | Real data flowing through all modules |
| Sprint 5 | Week 5 | Reports, CSV export, alerts wiring, polish | MVP complete and demo-ready |
| Sprint 6 | Week 6 | Docker, CI/CD, README, seed data, deployment | Deployed and publicly accessible |

---

## 13. Out of Scope

The following are explicitly excluded from v1 and will not be built unless the MVP is complete:

- Barcode scanner hardware integration
- Mobile native app (iOS / Android)
- ERP / accounting integration (QuickBooks, SAP, Xero)
- Multi-currency support
- Customer-facing portal
- Real-time WebSocket updates
- Offline / PWA support
- Advanced forecasting or demand planning
- Multi-tenancy (multiple companies in one instance)

---

## 14. Open Questions

| # | Question | Owner | Due |
|---|---|---|---|
| OQ-1 | Should stock levels be calculated on-the-fly from transactions, or maintained as a materialized `StockLevel` table? Materialized is faster to query but requires keeping it in sync. | Jeff | Sprint 3 kickoff |
| OQ-2 | What is the preferred CSV export format for reports — flat file or Excel-compatible with headers? | Jeff | Sprint 5 kickoff |
| OQ-3 | Should deleted products be soft-deleted (archived) or hard-deleted? Transactions reference products so hard delete would break history. | Jeff | Sprint 3 kickoff |
| OQ-4 | Is deployment to a public URL required for MVP, or is Docker Compose local-only sufficient for the portfolio? | Jeff | Sprint 6 kickoff |

---

*Document maintained in `/docs/TECHNICAL-DESIGN-DOCUMENT.md` in the project repository.*
