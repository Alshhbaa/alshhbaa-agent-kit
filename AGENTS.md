# AGENTS.md — Alshhbaa Copperware AI Agent Integration Guidelines

This document provides definitive instructions, protocols, and architectural guidelines for AI agents, autonomous bots, and LLM-powered shopping assistants interacting with **Alshhbaa Copperware** (متجر نحاسيات الشهباء).

---

## 1. Domain Context & Identity

- **Store:** Alshhbaa Copperware (متجر نحاسيات الشهباء)
- **Specialization:** Handcrafted brass lighting fixtures, heritage chandeliers, wall sconces, luxury table lamps, Bohemian crystal, and Italian glassware.
- **Base Domain:** `https://shop.alshhbaa.com`
- **Target Market:** Kingdom of Saudi Arabia (KSA) and GCC.
- **Primary Language:** Arabic (`ar-SA`). Catalog names, categories, and descriptions are primarily in Arabic.
- **Currency & Taxation:** Saudi Riyal (`SAR`). All consumer storefront prices and totals include 15% Value Added Tax (`VAT`), indicated by `priceIncludesVat: true`.

---

## 2. Safety, Governance & Human-in-the-Loop (HITL) Guardrails

All autonomous agents MUST adhere to the governance rules declared at `/.well-known/agent-permissions.json`:

### Mandatory Agent Directives
1. **Explicit Human Consent for Purchase:**
   - Agents **MUST NEVER** automatically place an order or initiate payment on behalf of a user without explicit confirmation of:
     - Selected items and options (finish, dimensions, variant).
     - Quantities and verified unit prices.
     - Destination city, address, and delivery carrier fee.
     - Final order total in SAR.
2. **Zero Payment Credential Handling:**
   - Agents **MUST NOT** request, store, or transmit payment credentials (credit/debit card numbers, CVVs, PINs, or bank passwords).
   - Checkout is conducted via the human-in-the-loop web flow at `https://shop.alshhbaa.com/checkout`.
3. **Admin Isolation:**
   - Agents **MUST NOT** attempt to access administrative routes (`/admin`, `/api/admin`) or attempt inventory modification.
4. **Data Authority:**
   - Agents **MUST NOT** hallucinate prices, availability, or delivery estimates. Rely strictly on live API responses.
5. **Customer Privacy:**
   - Order tokens (`token`) grant access to order status and invoices. Keep customer tokens strictly confidential.

---

## 3. Protocol Discovery Surfaces

Alshhbaa Copperware implements modern discovery and integration standards:

| Protocol / Spec | Location | Purpose |
|---|---|---|
| **OpenAPI 3.1** | `https://shop.alshhbaa.com/openapi.json` | Comprehensive REST API schema for all public endpoints. |
| **MCP (Streamable HTTP)** | `https://shop.alshhbaa.com/.well-known/mcp` | Remote Model Context Protocol server exposing tools and UI resources. |
| **Agent Plugins 1.0.0** | `plugin.json` & `mcp.json` | Manifests defining the kit and remote MCP server. |
| **Agent Skill** | `skills/alshhbaa-store/SKILL.md` | Structured skill instructions following `skills.sh`. |
| **Agent Permissions** | `https://shop.alshhbaa.com/.well-known/agent-permissions.json` | LAS-WG consent and permission rules. |
| **LLMs Context** | `https://shop.alshhbaa.com/llms.txt` | Curated index of store context and docs for LLMs. |
| **Agentic Web Brief** | `https://shop.alshhbaa.com/agenticweb.md` | Human/agent summary of capabilities and checkout limits. |
| **Developer Portal** | `https://shop.alshhbaa.com/developers` | Developer guides, API versioning, and webhook policies. |

---

## 4. API Architecture & Interaction Patterns

### Base URL & Versioning
- **Recommended Prefix:** `/api/v1/` (Stable API contract).
- Example: `https://shop.alshhbaa.com/api/v1/products`
- Legacy `/api/...` routes mirror `/api/v1/...` for backward compatibility.
- Any future breaking changes will follow a 90-day deprecation notice via RFC 9745 `Deprecation` and RFC 8594 `Sunset` headers.

### Standard Headers
- `Accept: application/json`
- `Content-Type: application/json` (for POST requests)
- `Idempotency-Key: <UUID>`: Recommended on all POST operations (`/api/v1/cart/sync`, `/api/v1/checkout`, `/api/v1/orders/{id}/status`). Length: 8-200 characters.

### Rate Limiting Handling
All API responses include rate limit headers:
- `RateLimit-Limit`: Request quota in current window.
- `RateLimit-Remaining`: Remaining request quota.
- `RateLimit-Reset`: Seconds remaining until window reset.
- If HTTP `429 Too Many Requests` is returned, the agent **MUST** wait for the seconds indicated in `Retry-After` before retrying.

---

## 5. Endpoints Reference for Agents

### 1. Product Search & Catalog Discovery

#### Fast Search Suggestions
- **Endpoint:** `GET /api/v1/search/suggestions`
- **Query Params:**
  - `q` (string, required): Search query (e.g., `ثريا`, `أباجورة`).
  - `category` (string, optional): Category slug.
  - `limit` (integer, optional, default: 5): Number of suggestions.
- **Use Case:** Interactive queries where quick product names and slugs are needed.

#### Paginated Catalog
- **Endpoint:** `GET /api/v1/products`
- **Query Params:**
  - `q` (string, optional): Keyword search.
  - `category` (string, optional): Category filter.
  - `limit` (integer, optional, 1-50, default: 20): Items per page.
  - `cursor` (string, optional): Opaque pagination cursor.
- **Response Format:**
  ```json
  {
    "products": [...],
    "total": 12,
    "pagination": {
      "limit": 20,
      "nextCursor": "eyJsYXN0SWQiOjEyfQ==",
      "hasMore": false
    }
  }
  ```
- **Rule:** When `hasMore` is `true`, pass `nextCursor` as `cursor` in the next call. The cursor is opaque; do not parse or mutate it.

### 2. Shipping & City Calculations

#### City Search
- **Endpoint:** `GET /api/v1/cities`
- **Query Params:**
  - `q` (string, optional): City name in Arabic (e.g. `الرياض`, `جدة`, `الدمام`).
  - `limit` (integer, optional): Maximum results.

#### Live Shipping Carriers
- **Endpoint:** `GET /api/v1/shipping-carriers`
- **Query Params:**
  - `city` (string, required): Target city name.
  - `productIds` (string, optional): Comma-separated product IDs to account for heavy or oversized brass fixtures.
- **Response:** List of available carriers, estimated delivery windows, and exact shipping prices in SAR.

### 3. Shopping Cart Validation

#### Sync Cart Items
- **Endpoint:** `POST /api/v1/cart/sync`
- **Header:** `Idempotency-Key: <uuid>`
- **Request Body:**
  ```json
  {
    "items": [
      {
        "productId": 42,
        "quantity": 2
      }
    ]
  }
  ```
- **Response:** Detailed line items with verified server-side unit prices, subtotal, tax amount, and any stock warnings if quantity exceeds available inventory.

### 4. Order Management & Invoices

#### Order Status & Fulfillment Tracking
- **Endpoint:** `GET /api/v1/orders/{id}/status?token={token}`
- **Security:** Requires customer-provided `token`.
- **Response:** Order status (`pending`, `confirmed`, `shipped`, `delivered`), payment status, tracking number, and tracking carrier URL when dispatched.

#### Asynchronous Invoice Generation
- **Endpoint:** `POST /api/v1/orders/{id}/invoice/jobs?token={token}`
- **Response:** `202 Accepted` with `Location` header and JSON body containing `jobId` and `statusUrl`.
- **Polling:** Query `GET /api/v1/orders/{id}/invoice/jobs/{jobId}?token={token}` until `status` becomes `succeeded`, then download via `downloadUrl` or with `download=1`.

---

## 6. Remote Model Context Protocol (MCP) Server

The remote MCP server operates at `https://shop.alshhbaa.com/.well-known/mcp` using the `streamable-http` transport (JSON-RPC 2.0).

### Available MCP Tools
- **`search_products`:**
  - Description: Search public catalog product suggestions.
  - Input: `{"query": string, "limit"?: integer}`
  - Annotations: `readOnlyHint: true`
- **`read_docs`:**
  - Description: Read store API and developer documentation.
  - Input: `{"path": string}` (one of `/developers`, `/llms.txt`, `/openapi.json`, `/auth.md`, `/versioning.md`, `/webhooks.md`, etc.)
  - Annotations: `readOnlyHint: true`

### Available MCP Resources
- `ui://alshhbaa/developer-portal.html`
- `ui://alshhbaa/catalog-search.html`

---

## 7. Typical Shopping Interaction Workflow

When an end-user instructs an agent: *"Find me a luxury brass chandelier under 3,000 SAR and calculate shipping to Riyadh"*:

1. **Search Catalog:**
   Call `GET /api/v1/products?q=ثريا&limit=20` to discover available brass chandeliers.
2. **Filter & Select:**
   Filter results where price <= 3000 SAR. Extract Arabic title, specifications, and dimensions.
3. **Verify City & Shipping:**
   Call `GET /api/v1/cities?q=الرياض` to ensure the city is valid.
   Call `GET /api/v1/shipping-carriers?city=الرياض&productIds={selected_id}` to retrieve carrier rates and delivery time.
4. **Synchronize Cart:**
   Call `POST /api/v1/cart/sync` with `items: [{productId, quantity: 1}]` to lock in current server prices and verify in-stock inventory.
5. **Present to User:**
   Present an Arabic/English summary:
   - Product name and specs.
   - Price breakdown (Product + 15% VAT included + Shipping).
   - Expected delivery timeline.
6. **Checkout Handoff:**
   Provide the user with the direct link to checkout: `https://shop.alshhbaa.com/checkout`.
