# Restaurant QR Order System — User Stories

## Overview

This document contains user stories for the Restaurant QR Order System, a table-based digital ordering platform for dine-in restaurants. It covers the full service funnel from customer QR Code scanning through kitchen preparation to table closure.

**User Types:**
- **Customer** — Anonymous diner scanning a QR Code at the table (no account required)
- **Waiter** — Front-of-house staff managing tables and delivering orders
- **Kitchen Operator** — Kitchen staff viewing and updating item preparation status
- **Owner** — Restaurant owner with full access to all management features

---

## 1. Tenant Registration & Authentication

### US-1.1: Restaurant Owner Registration
**As an** Owner  
**I want to** register my restaurant on the platform  
**So that** I can start managing orders and staff through my own tenant

**Acceptance Criteria:**
- [ ] Registration form collects: owner name, email, password, password confirmation, restaurant name
- [ ] Email must be unique across the platform
- [ ] Password must meet minimum security requirements (8+ characters)
- [ ] A tenant record is created and bound exclusively to the registering owner
- [ ] Owner is assigned the `owner` role automatically
- [ ] Owner is redirected to `/admin/dashboard` after successful registration
- [ ] All subsequent data created by this owner is scoped to their `tenant_id`

**Expected Result:** Restaurant tenant created and owner can begin configuring tables, menu, and staff.

---

### US-1.2: Staff Login
**As a** Waiter, Kitchen Operator, or Owner  
**I want to** log in with my email and password  
**So that** I can access my role-specific panel

**Acceptance Criteria:**
- [ ] Login form at `/admin/login` collects: email and password
- [ ] Failed login returns a non-revealing error message
- [ ] Rate limiting applied after repeated failed attempts
- [ ] On success, user is redirected based on role:
  - Owner / Waiter → `/admin/dashboard`
  - Kitchen Operator → `/kitchen`
- [ ] Session is tenant-scoped — no access to other tenants' data is possible

**Expected Result:** Staff member authenticated and redirected to the correct panel.

---

### US-1.3: Password Reset
**As a** Staff Member  
**I want to** reset my password via email  
**So that** I can regain access to my account if I forget my credentials

**Acceptance Criteria:**
- [ ] "Forgot password" link available on the login page
- [ ] Password reset email sent to the registered address
- [ ] Reset link is valid for a limited time (60 minutes)
- [ ] User can define and confirm a new password
- [ ] Old sessions invalidated after password change

**Expected Result:** Staff member securely regains account access.

---

## 2. Staff Management

### US-2.1: Create Staff Account
**As an** Owner  
**I want to** create accounts for waiters and kitchen operators  
**So that** my team can log in and perform their duties

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/staff`
- [ ] Form collects: name, email, password, role (Waiter or Kitchen Operator)
- [ ] New account is automatically scoped to the owner's `tenant_id`
- [ ] Staff member can log in immediately after creation
- [ ] Owner cannot create accounts with the `owner` role

**Expected Result:** Staff account created, scoped to the tenant, with appropriate role.

---

### US-2.2: Edit or Deactivate Staff Account
**As an** Owner  
**I want to** edit or deactivate a staff account  
**So that** I can manage team members as they join or leave

**Acceptance Criteria:**
- [ ] Owner can update name, email, and role of any staff member in their tenant
- [ ] Owner can deactivate an account (user can no longer log in)
- [ ] Deactivated accounts are not permanently deleted (history preserved)
- [ ] Owner cannot deactivate their own account

**Expected Result:** Staff account updated or blocked from access without data loss.

---

## 3. Table Management

### US-3.1: Create a Table
**As an** Owner  
**I want to** create a table record in the system  
**So that** customers can scan a QR Code and place orders at that table

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/tables`
- [ ] Form collects: table number, display name, seating capacity
- [ ] Table number must be unique within the tenant
- [ ] Table is created in `active` state by default
- [ ] A unique QR Code is automatically generated server-side upon creation
- [ ] QR Code encodes the URL `/order?table={number}`

**Expected Result:** Table created with a unique QR Code ready for placement.

---

### US-3.2: Download / Print QR Code
**As an** Owner  
**I want to** download or print the QR Code for a table  
**So that** I can place it physically on the table for customers to scan

**Acceptance Criteria:**
- [ ] QR Code available for download in PNG or SVG format
- [ ] Download option visible per table in the tables list
- [ ] QR Code is print-ready quality

**Expected Result:** QR Code file downloaded and ready for physical placement.

---

### US-3.3: Edit a Table
**As an** Owner  
**I want to** edit a table's details  
**So that** I can correct display names or adjust seating capacity

**Acceptance Criteria:**
- [ ] Owner can update: display name, seating capacity
- [ ] Table number cannot be changed after creation (it is encoded in the QR Code)
- [ ] Changes are saved immediately

**Expected Result:** Table details updated without affecting the QR Code URL.

---

### US-3.4: Deactivate a Table
**As an** Owner  
**I want to** deactivate a table  
**So that** customers cannot place orders at tables that are closed or unavailable

**Acceptance Criteria:**
- [ ] Owner can toggle a table between active and inactive
- [ ] When inactive, scanning the QR Code shows a friendly unavailability message instead of the menu
- [ ] Inactive tables are visually distinguished in the admin tables list

**Expected Result:** Inactive table blocks new orders while preserving its configuration.

---

## 4. Menu Management

### US-4.1: Manage Categories
**As an** Owner  
**I want to** create, edit, and reorder menu categories  
**So that** the menu is organized clearly for customers browsing on mobile

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/categories`
- [ ] Form collects: category name, display order
- [ ] Categories can be edited and deleted (deletion blocked if products are linked)
- [ ] Display order controls the sequence shown in the customer ordering screen
- [ ] Changes are immediately reflected in the live menu

**Expected Result:** Menu categories organized and ordered as the owner intends.

---

### US-4.2: Manage Products
**As an** Owner  
**I want to** create, edit, and manage products  
**So that** customers can browse and order items from the menu

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/products`
- [ ] Form collects: name, description, price, photo, category, availability toggle
- [ ] Photo upload supported (max 5MB, stored via Laravel Storage)
- [ ] Availability toggle hides/shows the product on the customer screen without deleting it
- [ ] Product slug is unique within the tenant (`tenant_id` + `product_slug` constraint)
- [ ] Products list is filterable by category

**Expected Result:** Product created or updated and visible on the live menu when available.

---

### US-4.3: Manage Extras (Paid Add-ons)
**As an** Owner  
**I want to** configure paid extras for specific products  
**So that** customers can customize their order with add-ons at an additional cost

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/extras`
- [ ] Form collects: extra name, price, linked product, required/optional flag
- [ ] Multiple extras can be linked to a single product
- [ ] Extras appear in the product customization modal on the ordering screen
- [ ] Required extras must be selected before adding to cart

**Expected Result:** Extras appear in the customization modal and are included in the order total.

---

### US-4.4: Manage Options (Free Customizations)
**As an** Owner  
**I want to** configure free options for specific products  
**So that** customers can specify preferences like cooking level or sauce choice

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/options`
- [ ] Form collects: option group name, list of choices, min and max selection rules, linked product
- [ ] Options with `min > 0` are mandatory before the item can be added to cart
- [ ] Options with no extra cost are clearly presented separately from paid extras

**Expected Result:** Options appear in the customization modal and are recorded with the order item.

---

### US-4.5: Manage Combos
**As an** Owner  
**I want to** assemble combo offers from existing products  
**So that** I can sell grouped items at a promotional price

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/combos`
- [ ] Form collects: combo name, description, list of included products, promotional price
- [ ] Combo promotional price is independent of individual product prices
- [ ] Combo appears as a single selectable item on the customer ordering screen
- [ ] Combos can be activated or deactivated without deletion

**Expected Result:** Combo available on the menu as a single item at the set promotional price.

---

## 5. Customer Ordering Screen

### US-5.1: Scan QR Code and Load Menu
**As a** Customer  
**I want to** scan the QR Code on my table and see the menu immediately  
**So that** I can browse and order without downloading an app or creating an account

**Acceptance Criteria:**
- [ ] Scanning QR Code opens `/order?table={number}` in the mobile browser
- [ ] System validates that the table is active
- [ ] If table is inactive, a friendly unavailability message is shown
- [ ] Full menu loads (categories, products, combos) without requiring login
- [ ] Page is mobile-first and usable on low-end Android devices

**Expected Result:** Customer sees the full menu immediately after scanning, with no friction.

---

### US-5.2: Browse Menu by Category
**As a** Customer  
**I want to** navigate the menu by category  
**So that** I can quickly find what I'm looking for

**Acceptance Criteria:**
- [ ] Category tabs displayed as a horizontally scrollable row at the top
- [ ] Tapping a category scrolls or filters to its products
- [ ] Each product card shows: photo, name, description, and price
- [ ] Unavailable products are hidden from the menu

**Expected Result:** Customer can find and view products organized by category.

---

### US-5.3: Customize a Product and Add to Cart
**As a** Customer  
**I want to** configure extras and options before adding a product to my cart  
**So that** my order matches exactly what I want

**Acceptance Criteria:**
- [ ] Tapping a product card opens a customization modal
- [ ] Modal shows all configured extras (paid) and options (free) for that product
- [ ] Running total updates dynamically with each selection
- [ ] Required extras and options must be selected before the "Add to Cart" button is enabled
- [ ] Customer can set item quantity in the modal
- [ ] Item added to the cart (client-side JS state, no server round-trip until confirmation)

**Expected Result:** Item added to cart with all customizations and correct subtotal.

---

### US-5.4: Review Cart and Confirm Order
**As a** Customer  
**I want to** review my cart and confirm my order  
**So that** the kitchen receives my selections and begins preparation

**Acceptance Criteria:**
- [ ] Sticky bottom bar shows item count and subtotal at all times
- [ ] Tapping the bar opens the cart review screen with all items and their configurations
- [ ] Customer can remove items from the cart before confirming
- [ ] "Confirm Order" submits a POST to the backend with `table_id` and all item configurations
- [ ] On success: confirmation message shown; cart cleared
- [ ] On failure: error message shown with option to retry
- [ ] No authentication required for this action

**Expected Result:** Order submitted to the system and kitchen receives it in real time.

---

### US-5.5: Place Additional Orders at the Same Table
**As a** Customer  
**I want to** place more orders later during my meal  
**So that** I can order drinks or dessert without flagging down a waiter

**Acceptance Criteria:**
- [ ] Customer can scan the QR Code again at any point during the session
- [ ] New orders are added to the same open table session
- [ ] Running total on the dashboard accumulates all orders from the session
- [ ] Each new order batch is timestamped and visible to staff separately

**Expected Result:** Additional orders appended to the active table session seamlessly.

---

## 6. Kitchen Panel

### US-6.1: View Incoming Orders in Real Time
**As a** Kitchen Operator  
**I want to** see new orders appear on my screen the moment they are confirmed  
**So that** I can start preparation without delay

**Acceptance Criteria:**
- [ ] Kitchen Operator accesses `/kitchen` (route protected — kitchen role only)
- [ ] Livewire v4 component updates the order queue in real time without page refresh
- [ ] Each order card displays: table number, item name, extras, options, customer notes, arrival time
- [ ] Orders sorted by arrival time (oldest first)
- [ ] Visual alert shown for orders waiting longer than a configurable threshold

**Expected Result:** Kitchen sees new orders instantly and can prioritize by wait time.

---

### US-6.2: Mark Item as In Preparation
**As a** Kitchen Operator  
**I want to** mark an item as "In Preparation"  
**So that** the table dashboard reflects that the kitchen has started working on it

**Acceptance Criteria:**
- [ ] "In Preparation" button visible on each order card in `Waiting` status
- [ ] One tap updates item status to `In Preparation`
- [ ] Waiter's table dashboard reflects the change reactively in real time
- [ ] Card visual style updated to distinguish it from items still waiting

**Expected Result:** Item status transitions from Waiting to In Preparation across all panels.

---

### US-6.3: Mark Item as Ready
**As a** Kitchen Operator  
**I want to** mark a completed item as "Ready"  
**So that** waiters know the item is plated and can be picked up for delivery

**Acceptance Criteria:**
- [ ] "Ready" button visible on each order card in `In Preparation` status
- [ ] One tap updates item status to `Ready`
- [ ] Item flagged visually on the waiter's table dashboard for pickup
- [ ] Ready items are visually distinct from In Preparation items on both screens

**Expected Result:** Waiter is alerted that the item is ready for delivery.

---

## 7. Table Dashboard

### US-7.1: View All Tables with Status Indicators
**As an** Owner or Waiter  
**I want to** see all tables on a single dashboard with color-coded status  
**So that** I can instantly understand the state of the floor

**Acceptance Criteria:**
- [ ] Owner or Waiter accesses `/admin/dashboard`
- [ ] All tables displayed in a responsive Bootstrap grid
- [ ] Color coding applied:
  - Green — table has active orders in progress
  - Yellow — customer is currently placing an order via QR Code
  - Gray — table is free/idle
- [ ] Aggregate metrics shown at the top:
  - Total active tables
  - Items awaiting delivery
  - Revenue for the current day
- [ ] Dashboard updates reactively via Livewire v4 without page refresh

**Expected Result:** Staff has a real-time overview of the entire floor at a glance.

---

### US-7.2: View Table Detail and Order History
**As an** Owner or Waiter  
**I want to** click a table and see all open orders for that session  
**So that** I can track what was ordered, what has been delivered, and the running total

**Acceptance Criteria:**
- [ ] Clicking a table card opens a detail view (Livewire modal)
- [ ] Detail view shows:
  - Full order list for the current session (all batches)
  - Per-item status: Waiting / In Preparation / Ready / Delivered
  - Accumulated total for the session
- [ ] View updates in real time as kitchen changes item statuses

**Expected Result:** Staff has complete visibility into a table's session at any moment.

---

### US-7.3: Mark Item as Delivered
**As a** Waiter  
**I want to** mark an item as delivered once I bring it to the table  
**So that** the order status is accurate and the kitchen queue is cleared

**Acceptance Criteria:**
- [ ] "Delivered" action available for items in `Ready` status from the table detail view
- [ ] One tap updates item status to `Delivered`
- [ ] Item moves out of the kitchen queue and is recorded as delivered with a timestamp

**Expected Result:** Item marked as delivered and removed from the pending delivery list.

---

### US-7.4: Close a Table
**As a** Waiter or Owner  
**I want to** close a table after the bill is settled  
**So that** the session is finalized and the table is marked as free for new customers

**Acceptance Criteria:**
- [ ] "Close Table" action available in the table detail view
- [ ] Confirmation prompt shown before closing
- [ ] On confirm:
  - Session finalized with final total recorded
  - All orders archived for reporting
  - Table status reset to idle (gray)
- [ ] Closed session is preserved in reports

**Expected Result:** Table session finalized, table freed, and data archived for reporting.

---

### US-7.5: Reopen a Recently Closed Table
**As a** Waiter or Owner  
**I want to** reopen a table that was just closed by mistake  
**So that** I can continue the session without losing the order history

**Acceptance Criteria:**
- [ ] "Reopen" action available on recently closed tables
- [ ] Reopening restores the previous session's orders and accumulated total
- [ ] Table status returns to active (green)

**Expected Result:** Table session restored without data loss.

---

## 8. Reports

### US-8.1: View Daily Operations Report
**As an** Owner  
**I want to** view a report of all orders and revenue for a given day  
**So that** I can monitor business performance

**Acceptance Criteria:**
- [ ] Owner accesses `/admin/reports`
- [ ] Default view shows the current day's data
- [ ] Report includes:
  - Total revenue for the period
  - Number of tables served
  - Number of orders placed
  - Best-selling products (name + quantity sold)
  - Per-table session totals

**Expected Result:** Owner has a clear summary of daily performance.

---

### US-8.2: Filter Report by Date Range
**As an** Owner  
**I want to** filter the report by a custom date range  
**So that** I can analyze performance across specific periods (e.g., a week or month)

**Acceptance Criteria:**
- [ ] Date range picker available on the reports page
- [ ] Report data refreshes based on selected start and end dates
- [ ] Metrics recalculated for the selected range

**Expected Result:** Report reflects only the selected date range.

---

### US-8.3: Export Report as CSV
**As an** Owner  
**I want to** export the report data as a CSV file  
**So that** I can share it with my accountant or analyze it in a spreadsheet

**Acceptance Criteria:**
- [ ] "Export CSV" button available on the reports page
- [ ] Export respects the currently applied date range filter
- [ ] CSV includes: table number, session total, item list, timestamp per session
- [ ] Best-selling products included as a summary section or separate sheet

**Expected Result:** CSV file downloaded with complete report data for the selected period.

---

## Appendix: User Story Status

| ID | Story | Role | Priority | Status |
|----|-------|------|----------|--------|
| US-1.1 | Restaurant Owner Registration | Owner | High | Pending |
| US-1.2 | Staff Login | All Staff | High | Pending |
| US-1.3 | Password Reset | All Staff | Medium | Pending |
| US-2.1 | Create Staff Account | Owner | High | Pending |
| US-2.2 | Edit or Deactivate Staff Account | Owner | Medium | Pending |
| US-3.1 | Create a Table | Owner | High | Pending |
| US-3.2 | Download / Print QR Code | Owner | High | Pending |
| US-3.3 | Edit a Table | Owner | Low | Pending |
| US-3.4 | Deactivate a Table | Owner | Medium | Pending |
| US-4.1 | Manage Categories | Owner | High | Pending |
| US-4.2 | Manage Products | Owner | High | Pending |
| US-4.3 | Manage Extras | Owner | Medium | Pending |
| US-4.4 | Manage Options | Owner | Medium | Pending |
| US-4.5 | Manage Combos | Owner | Low | Pending |
| US-5.1 | Scan QR Code and Load Menu | Customer | High | Pending |
| US-5.2 | Browse Menu by Category | Customer | High | Pending |
| US-5.3 | Customize Product and Add to Cart | Customer | High | Pending |
| US-5.4 | Review Cart and Confirm Order | Customer | High | Pending |
| US-5.5 | Place Additional Orders | Customer | Medium | Pending |
| US-6.1 | View Incoming Orders in Real Time | Kitchen Operator | High | Pending |
| US-6.2 | Mark Item as In Preparation | Kitchen Operator | High | Pending |
| US-6.3 | Mark Item as Ready | Kitchen Operator | High | Pending |
| US-7.1 | View All Tables with Status Indicators | Owner / Waiter | High | Pending |
| US-7.2 | View Table Detail and Order History | Owner / Waiter | High | Pending |
| US-7.3 | Mark Item as Delivered | Waiter | High | Pending |
| US-7.4 | Close a Table | Waiter / Owner | High | Pending |
| US-7.5 | Reopen a Recently Closed Table | Waiter / Owner | Medium | Pending |
| US-8.1 | View Daily Operations Report | Owner | Medium | Pending |
| US-8.2 | Filter Report by Date Range | Owner | Medium | Pending |
| US-8.3 | Export Report as CSV | Owner | Low | Pending |
