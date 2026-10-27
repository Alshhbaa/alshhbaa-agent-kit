---
name: alshhbaa-store
description: Search Alshhbaa Brassware luxury lighting catalog, check real-time stock and prices, calculate shipping rates across Saudi cities, and validate shopper carts.
---

# Alshhbaa Brassware Store Skill

Use this skill when an AI agent needs to browse, search, validate carts, or retrieve store information for **Alshhbaa Brassware** (متجر نحاسيات الشهباء), an authentic Saudi retailer specializing in handcrafted brass chandeliers, wall sconces, luxury table lamps, Bohemian crystal, and Italian glassware.

## Base URLs and Authentication

- **Production Storefront & API:** `https://shop.alshhbaa.com`
- **MCP Server (Streamable HTTP):** `https://shop.alshhbaa.com/.well-known/mcp`
- **OpenAPI 3.1 Specification:** `https://shop.alshhbaa.com/openapi.json`
- **Developer Documentation:** `https://shop.alshhbaa.com/developers`
- **Agent Governance:** `https://shop.alshhbaa.com/.well-known/agent-permissions.json`
- **Authentication:** Zero-auth for public browsing, search, catalog, city lookups, and cart validation. Order status and invoices require a customer-provided order access token.

## Capabilities

1. **Catalog Search & Suggestions:** Fast auto-complete and full product listing with cursor-based pagination.
2. **Real-time Price & Stock Verification:** Always verify current stock levels and prices directly from the API.
3. **Saudi City & Shipping Rates:** Search supported cities and obtain live carrier delivery quotes.
4. **Cart Synchronization & Price Validation:** Validate item IDs, quantities, and selected options against live server prices.
5. **Order Tracking:** Retrieve shipment and order progress using customer tokens.
6. **MCP Remote Server Integration:** Invoke MCP tools (`search_products`, `read_docs`) over streamable HTTP.

---

## API Operations & curl Examples

All endpoints accept standard HTTP requests. Versioned endpoints (`/api/v1/...`) are recommended for stability.

### 1. Fast Product Search Suggestions

Search products with Arabic or English search terms without querying the entire catalog database.

```bash
curl -s -X GET "https://shop.alshhbaa.com/api/v1/search/suggestions?q=%D8%AB%D8%B1%D9%8A%D8%A7" \
  -H "Accept: application/json"
```

### 2. Product Catalog Listing & Cursor Pagination

List published products with opaque cursor-based pagination (`nextCursor`).

```bash
# First page
curl -s -X GET "https://shop.alshhbaa.com/api/v1/products?limit=10" \
  -H "Accept: application/json"

# Subsequent page using nextCursor
curl -s -X GET "https://shop.alshhbaa.com/api/v1/products?limit=10&cursor=YOUR_NEXT_CURSOR" \
  -H "Accept: application/json"
```

Response includes `products`, `total`, and `pagination` (`limit`, `nextCursor`, `hasMore`). Prices are in SAR and include 15% VAT (`priceIncludesVat: true`).

### 3. City Search for Shipping

Search supported Saudi Arabian cities for delivery quotes.

```bash
curl -s -X GET "https://shop.alshhbaa.com/api/v1/cities?q=%D8%A7%D9%84%D8%B1%D9%8A%D8%A7%D8%B6" \
  -H "Accept: application/json"
```

### 4. Live Shipping Carrier Quotes

Fetch real-time shipping carrier availability and rates for a specified destination city.

```bash
curl -s -X GET "https://shop.alshhbaa.com/api/v1/shipping-carriers?city=%D8%A7%D9%84%D8%B1%D9%8A%D8%A7%D8%B6" \
  -H "Accept: application/json"
```

### 5. Cart Synchronization & Server-Side Price Verification

Synchronize user cart items to calculate verified totals, check available inventory, and prevent stale client prices. Always send an `Idempotency-Key` header on write requests.

```bash
curl -s -X POST "https://shop.alshhbaa.com/api/v1/cart/sync" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d" \
  -d '{
    "items": [
      {
        "productId": 42,
        "quantity": 1
      }
    ]
  }'
```

### 6. Order Status & Tracking

Retrieve order status, fulfillment status, and carrier tracking links using the customer-provided order ID and access token.

```bash
curl -s -X GET "https://shop.alshhbaa.com/api/v1/orders/101/status?token=CUSTOMER_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

### 7. MCP Streamable HTTP JSON-RPC Request

Interact with the official remote MCP server over HTTP JSON-RPC 2.0.

```bash
# List available MCP tools
curl -s -X POST "https://shop.alshhbaa.com/.well-known/mcp" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list"
  }'

# Initialize session
curl -s -X POST "https://shop.alshhbaa.com/.well-known/mcp" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05"
    }
  }'
```

---

## Agent Decision & Operational Rules

1. **Product Data Authority:** Always treat data returned by `/api/v1/products` or live product pages under `https://shop.alshhbaa.com/products/{slug}` as authoritative. Never infer or guess stock availability, discounts, or delivery times.
2. **Arabic-First Storefront:** Preserve Arabic names and titles for items, categories, and options (e.g. "ثريا نحاس كلاسيك", "أباجورة بوهيمي") when presenting choices to users.
3. **Strict Human-in-the-Loop Checkout:**
   - AI agents **MUST NOT** complete payment or charge payment methods autonomously.
   - AI agents **MUST NOT** collect, store, or transmit payment credentials (card numbers, CVVs).
   - Checkout is completed by the shopper in their browser at `https://shop.alshhbaa.com/checkout`.
   - Provide the customer with an itemized summary (products, quantities, shipping fee, total in SAR with VAT) and direct them to the checkout page.
4. **Rate Limiting & Retries:**
   - Respect `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` headers.
   - On HTTP 429 responses, pause execution for the duration specified in the `Retry-After` header before retrying.
5. **Private Data Protection:** Order tokens and customer delivery information are private. Never publish customer tokens or attempt unauthorized access.
