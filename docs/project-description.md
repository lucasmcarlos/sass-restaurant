# Restaurant QR Order System — Project Description

---

## Overview

We are building an MVP of a **table-based digital ordering platform** for dine-in restaurants. The platform allows restaurant owners to register their business, manage staff (waiters and kitchen operators), and control the full service funnel — from the moment a customer scans a QR Code at the table to the final bill closure.

The heart of the system is the **Table Dashboard (Kanban-style)**. Each physical table has a unique QR Code. When a customer scans it, they are taken directly to the digital menu for that table — no app installation required. Orders flow through a standardized status pipeline visible in real time to all staff.

**Standard order status pipeline:**
1. **Waiting** — order submitted by the customer, not yet acknowledged
2. **In Preparation** — kitchen has started working on the item
3. **Ready** — item is plated and waiting for pickup
4. **Delivered** — waiter has delivered the item to the table
5. **Closed** — table session finalized and bill settled

Clicking a table card on the dashboard opens a detailed view where staff can see all open orders for that table, item statuses, accumulated total, and manage the table session.

---

### 📋 Menu & Order Composition

The menu is organized in a flexible hierarchy optimized for both the admin panel and the customer-facing ordering screen:

- **Categories** — group products (e.g., Starters, Mains, Drinks, Desserts) with display order control.
- **Products** — each item has a name, description, price, photo, category, and availability toggle.
- **Extras** — paid add-ons attached to specific products (e.g., extra bacon +R$4.00, double cheese +R$3.00). Can be marked as required or optional.
- **Options** — free customization choices attached to specific products with min/max selection rules (e.g., cooking preference: rare / medium / well done).
- **Combos** — groups of products sold together at a promotional price (e.g., Family Combo = Burger + Fries + Drink for R$49.90).

When a customer selects a product on the ordering screen, the system presents its configured extras and options before adding it to the cart.

---

### 🪑 QR Code & Table Management

Each table is managed by the admin and assigned a unique QR Code generated server-side. The QR Code encodes the URL `/order?table={number}`. The admin panel allows:

- Creating, editing, and deactivating tables.
- Generating and downloading/printing QR Codes per table.
- Setting table capacity and display name.

When a table is inactive, scanning its QR Code shows a friendly unavailability message instead of the menu. Active table sessions accumulate all orders placed from that table until the waiter closes it.

---

### 🌐 Customer Ordering Screen

The customer-facing interface is a mobile-first PWA (Progressive Web App) rendered via Blade + Bootstrap 5. It requires no login or account creation. Features include:

- **Category navigation** — horizontal scroll tabs for fast browsing on mobile.
- **Product cards** — photo, name, description, and price. Tap to open the customization modal.
- **Extras & Options modal** — dynamically loaded per product. Running total recalculated on each selection.
- **Cart summary** — sticky bottom bar showing item count and subtotal. One tap to review and confirm.
- **Order confirmation** — sends the order via POST to the Laravel backend with `table_id` and all item configurations. No authentication required for this action.

---

### 🍳 Kitchen Panel

A dedicated real-time screen for kitchen staff, optimized for tablets or mounted monitors. Powered by Livewire v4 for instant updates without page refresh. Features:

- Order queue sorted by arrival time.
- Cards showing table number, items, extras, options, and any customer notes.
- One-tap status transitions: **Waiting → In Preparation → Ready**.
- Visual alerts for orders waiting longer than a configurable threshold.

---

### 📊 Table Dashboard (Admin & Waiter)

The main operational screen for owners and waiters, built with Livewire v4. Displays all tables in a responsive Bootstrap grid with color-coded status indicators:

- 🟢 **Green** — table has active orders in progress.
- 🟡 **Yellow** — customer is currently placing an order via QR Code.
- ⚪ **Gray** — table is free/idle.

Clicking a table opens a detail view with:
- Full order history for the current session.
- Per-item status tracking.
- Accumulated total.
- Actions: mark items as delivered, close the table, reopen if needed.

Aggregate metrics displayed at the top of the dashboard:
- Total active tables | Orders awaiting delivery | Revenue for the current service period.

---

## Tech Stack

| Layer | Technology | Justification |
|---|---|---|
| Backend / Framework | **Laravel 13** | Robust PHP framework with native routing, Eloquent ORM, queues, and middleware pipeline. Ideal for structured CRUD and multi-role auth. |
| Frontend / Templates | **Blade + Bootstrap 5** | Blade is Laravel's native templating engine. Bootstrap delivers responsive layout and ready-made UI components without SPA overhead. |
| Interactivity | **Livewire v4** | Reactive UI components for the admin panel (table dashboard, CRUD modals, real-time kitchen updates) without writing a separate JS framework. |
| Database | **PostgreSQL 18** | Robust relational database with strong transactional guarantees, JSONB support for flexible extras/options data, and excellent query performance. |
| QR Code Generation | **simplesoftwareio/simple-qrcode** | Laravel package for server-side QR Code generation in PNG or SVG. Each QR encodes `/order?table=N`. |
| File Storage | **Laravel Storage (local/S3-compatible)** | Handles product photo uploads. Configurable to local disk for MVP, easily switchable to S3 or compatible object storage in production. |
| Multi-tenancy | **Single-database, `tenant_id` scoping** | All records are scoped by `tenant_id` enforced via a shared Eloquent trait with Global Scope on all models. No extra packages required. |
| Deploy | **Laravel Forge + VPS** or **Railway** | Forge automates server provisioning, SSL, and Git-based deployments. Railway is a simpler alternative for smaller setups. |

---

### Application Architecture

```
Customer (mobile browser)
    │
    │  Scans QR Code → /order?table=N
    ▼
[ Blade + Bootstrap 5 (PWA) ]  ◄──►  [ Laravel 13 (Controllers, Eloquent, Policies) ]
                                                  │
                                    ┌─────────────┼──────────────┐
                                    ▼             ▼              ▼
                             [ PostgreSQL 18 ] [ Livewire v4 ] [ Laravel Storage ]
                             (single DB,       (admin panel,   (product photos)
                              tenant-scoped)    kitchen panel)
```

- Laravel serves all routes: admin (`/admin/*`), kitchen (`/kitchen`), waiter views, and customer ordering (`/order/*`).
- Livewire v4 powers all reactive admin screens: table dashboard, CRUD modals, kitchen queue, and live status updates.
- The customer ordering screen is a lightweight Blade PWA — no Livewire dependency, optimized for mobile load speed on low-end devices.
- PostgreSQL is accessed exclusively through Eloquent with mandatory `tenant_id` Global Scope enforced on all models.

---

### Route Structure

```
/                            → Redirects to /admin/login
/order?table={number}        → Customer menu screen (Blade PWA, public/anonymous)
/admin/login                 → Staff login
/admin/dashboard             → Table dashboard (owner/waiter)
/admin/tables                → Tables CRUD + QR Code generation
/admin/products              → Products CRUD
/admin/categories            → Categories CRUD
/admin/extras                → Extras CRUD
/admin/options               → Options CRUD
/admin/combos                → Combos CRUD
/admin/orders/{table}        → Order detail view per table
/admin/reports               → Daily/period reports
/admin/staff                 → Staff account management (owner only)
/kitchen                     → Kitchen real-time panel (kitchen role)
```

---

## Users & Roles (RBAC)

**Restaurant Owner (1–3 users per tenant):** Full access to all data within their own restaurant tenant. Can view and manage all orders, tables, products, combos, staff accounts, and reports. Can open and close tables, generate QR Codes, configure the full menu, and view overall performance metrics.

**Waiter / Front-of-House Staff (2–20+ users per tenant):** Can view the table dashboard and all active orders. Can mark items as delivered and close tables. Cannot access product/menu management, staff settings, or financial reports.

**Kitchen Operator (1–10+ users per tenant):** Access limited exclusively to the kitchen panel. Can view incoming orders in real time and update item statuses (In Preparation → Ready). Cannot access the table dashboard, menu settings, or any financial data.

---

## Core Workflows

### 1. Customer Order Flow

```
1. Customer scans the QR Code on the table
       │
       ▼
2. Laravel reads the ?table=N URL parameter
   → Validates that the table is active
   → Loads the menu via Eloquent (categories, products, extras, options, combos)
   → Renders the Blade PWA with the full menu (no auth required)
       │
       ▼
3. Customer browses the menu on mobile (Bootstrap responsive layout)
   → Selects a product
   → Configures extras and options in the customization modal
   → Dynamic subtotal recalculated on each selection
   → Adds item to cart (JS cart state, no server round-trip until confirmation)
       │
       ▼
4. Customer reviews cart and confirms order
   → POST /order/confirm with table_id and items[] (anonymous, no authentication)
   → Laravel creates records in `orders` and `order_items` tables (scoped to tenant_id)
   → Livewire components on kitchen panel and dashboard receive the update reactively
       │
       ▼
5. Kitchen receives the order in real time
   → New order card appears in the Livewire kitchen queue instantly
   → Kitchen operator marks item: In Preparation → Ready
       │
       ▼
6. Waiter collects and delivers the item
   → Marks item as Delivered on the table dashboard
       │
       ▼
7. Customer may place additional orders at any time
   → All orders accumulate in the same table session with a running total
       │
       ▼
8. Waiter closes the table
   → System records final total, archives all orders, resets table status to idle
```

---

### 2. Admin Setup Workflow

```
1. Owner registers at /register
   → Provides restaurant name and personal details
   → Becomes the first admin user of the tenant
   → A tenant record is created and the owner is strictly bound to it

2. Table configuration — /admin/tables
   → Creates tables with number, display name, and seating capacity
   → System generates a unique QR Code per table (simplesoftwareio/simple-qrcode)
   → QR Code encodes: https://domain.com/order?table=N
   → Admin downloads or prints QR Codes for physical placement on tables

3. Menu setup
   → Creates categories with display order (/admin/categories)
   → Adds products with name, description, price, photo, and category (/admin/products)
   → Configures extras (paid add-ons) and links them to specific products (/admin/extras)
   → Configures options (free choices) with min/max rules per product (/admin/options)
   → Assembles combos by grouping existing products at a promotional price (/admin/combos)

4. Staff account creation — /admin/staff
   → Owner creates accounts for waiters and kitchen operators
   → Each staff member is assigned a role and strictly scoped to the owner's tenant
   → Staff members log in at /admin/login with their credentials

5. Menu goes live
   → Customers can immediately start ordering by scanning any active table's QR Code
```

---

### 3. Kitchen Workflow

```
1. Kitchen operator logs in and accesses /kitchen
   → Route protected by middleware: only users with the kitchen role are admitted

2. Livewire v4 component maintains a live reactive connection
   → New orders appear instantly without page refresh or polling

3. Order queue displayed as Bootstrap cards sorted by arrival time (oldest first)
   → Each card shows: table number, items, extras, options, and customer notes

4. Kitchen operator taps "In Preparation"
   → Laravel updates item status via Livewire action
   → Table dashboard for waiters reflects the change reactively in real time

5. Kitchen operator taps "Ready"
   → Item is flagged on the waiter's dashboard for pickup and delivery
   → Visual indicator distinguishes Ready items from In Preparation items
```

---

### 4. Table Dashboard Workflow

```
1. Owner or waiter logs in and accesses /admin/dashboard

2. Livewire v4 renders a Bootstrap grid of all tables with color-coded status:
   → Green  = table has active orders in progress
   → Yellow = customer is currently placing an order via QR Code
   → Gray   = table is free/idle

3. Clicking a table opens a detail view (Livewire modal) showing:
   → Full order list for the current session
   → Per-item status: Waiting / In Preparation / Ready / Delivered
   → Accumulated total for the session

4. Waiter actions available from the detail view:
   → Mark individual items as Delivered
   → Close the table (finalizes session, resets table status to idle)
   → Reopen a recently closed table if needed

5. Aggregate metrics shown at the top of the dashboard:
   → Active tables | Items awaiting delivery | Revenue for the current day

6. Daily report — /admin/reports
   → Filterable by date range
   → Exportable CSV with per-table totals and best-selling products
```

---

## Technical Requirements (Production-Ready)

### Secure Authentication + Password Reset**
- Production-grade login with rate limiting, session security, and password reset flows.
- Tenant-aware authentication: users belong strictly to one tenant and cannot access or view data from any other tenant.
- Role-based route protection enforced via Laravel middleware and Policies for every route and Livewire action.

### Strict Single-Database Multi-Tenancy
- Every database record must be associated with a `tenant_id`.
- Tenant isolation enforced at the architecture level, not per-controller.
- **Mandatory approach:** A shared trait applied to all Eloquent models that:
  - Applies a **Global Scope** to automatically filter all queries by `tenant_id`.
  - Automatically injects `tenant_id` on model creation.
  - Prevents cross-tenant access, updates, or deletions at the query builder layer.
- Unique constraints enforced at the database level: `tenant_id` + `table_number`, `tenant_id` + `product_slug`.

### RBAC Enforced on the Backend
- Server-side permissions for all actions (view / create / update / delete) via Laravel Policies.
- Kitchen operators access only the kitchen panel — no visibility into financials or menu management.
- Waiters access only operational views — no access to menu configuration or staff management.
- Owners have unrestricted access within their own tenant.
- Customer ordering is fully anonymous — no authentication required, scoped only by active `table_id`.

### Mobile-First UI for Floor Operations
- Highly responsive interface for waiter and kitchen workflows (dashboard, order cards, status transitions).
- Optimized for mobile and tablet devices: fast navigation, readable cards, easy tap targets.
- Customer ordering screen (Blade PWA) designed exclusively for mobile — minimal JS, fast load, works reliably on low-end Android devices commonly found in restaurant environments.
- Admin CRUD screens designed for desktop/tablet with Bootstrap 5 responsive grid.