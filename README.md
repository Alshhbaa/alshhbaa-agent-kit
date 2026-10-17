# Alshhbaa Copperware AI Agent Kit 🏺✨

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Agent Plugins](https://img.shields.io/badge/Agent_Plugins-1.0.0-blue.svg)](https://agent-plugins.org)
[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.1.0-green.svg)](https://shop.alshhbaa.com/openapi.json)
[![MCP](https://img.shields.io/badge/MCP-streamable--http-orange.svg)](https://shop.alshhbaa.com/.well-known/mcp)
[![Smithery](https://img.shields.io/badge/Smithery-alshhbaa--r%2Fstore-blue.svg)](https://smithery.ai/servers/alshhbaa-r/store)
[![skills.sh](https://img.shields.io/badge/skills.sh-alshhbaa--store-blue.svg)](https://skills.sh/alshhbaa/alshhbaa-agent-kit)

The official AI Agent Integration Kit for **[Alshhbaa Copperware (متجر نحاسيات الشهباء)](https://shop.alshhbaa.com)** — a premier Saudi artisan retailer specializing in handcrafted brass chandeliers, Islamic & heritage lighting, wall sconces, luxury table lamps, Bohemian crystal, and Italian glassware.

This repository provides everything autonomous AI agents, coding assistants, and LLMs (Claude, ChatGPT, Cursor, Google Antigravity, OpenClaw, Cline) need to discover, query, validate, and interact with the Alshhbaa Copperware store safely and efficiently.

---

## 📦 What's Inside

```
alshhbaa-agent-kit/
├── README.md                          # Main documentation and integration overview
├── AGENTS.md                          # Detailed instructions & guidelines for AI agents
├── plugin.json                        # Agent Plugins 1.0.0 manifest
├── mcp.json                           # MCP remote streamable HTTP server configuration
├── LICENSE                            # MIT License
├── .gitignore                         # Standard git ignore rules
└── skills/
    └── alshhbaa-store/
        └── SKILL.md                   # Agent skill definition conforming to skills.sh
```

---

## ⚡ Quickstart

### 1. Using with Model Context Protocol (MCP)

Alshhbaa Copperware provides a remote streamable HTTP MCP server at:
```
https://shop.alshhbaa.com/.well-known/mcp
```

#### In Claude Desktop / Cline / Cursor (`claude_desktop_config.json` or `mcp.json`):
```json
{
  "mcpServers": {
    "alshhbaa-store": {
      "type": "streamable-http",
      "url": "https://shop.alshhbaa.com/.well-known/mcp"
    }
  }
}
```

The server provides:
- **`search_products`**: Fast public catalog search suggestions for chandeliers, lamps, and glassware.
- **`read_docs`**: Access to store developer guides, OpenAPI specifications, and policy documents.
- **UI Resources**: Interactive HTML components for developer portals and catalog search surfaces.

### 2. Using with Agent Plugins & Skills (`skills.sh`)

Install or load the `alshhbaa-store` skill directly:
- **Skill Path:** [`skills/alshhbaa-store/SKILL.md`](skills/alshhbaa-store/SKILL.md)
- **Plugin Manifest:** [`plugin.json`](plugin.json)

The skill instructs agents how to browse products, check Saudi shipping carrier fees, and validate shopping carts before handoff to checkout.

### 3. Direct REST API & OpenAPI 3.1

The complete OpenAPI specification is accessible at:
```
https://shop.alshhbaa.com/openapi.json
```

Key REST API endpoints:

| Endpoint | Method | Description | Auth |
|---|---|---|---|
| `/api/v1/products` | `GET` | Cursor-paginated catalog listing with filters | None (Public) |
| `/api/v1/search/suggestions` | `GET` | Fast suggestions search by query `?q=` | None (Public) |
| `/api/v1/cities` | `GET` | Search supported Saudi cities for shipping | None (Public) |
| `/api/v1/shipping-carriers` | `GET` | Live shipping quotes based on city & items | None (Public) |
| `/api/v1/cart/sync` | `POST` | Server-side cart validation and stock checking | None (`Idempotency-Key`) |
| `/api/v1/orders/{id}/status` | `GET` | Order fulfillment status & shipment tracking | Customer Token |
| `/api/v1/orders/{id}/invoice/jobs` | `POST` | Asynchronous PDF invoice generation | Customer Token |

---

## 🛡️ Governance & Human-in-the-Loop (HITL) Guardrails

Alshhbaa Copperware enforces strict agent permissions as defined in `/.well-known/agent-permissions.json`:

1. **Human-in-the-Loop Checkout:** AI agents are allowed to search, browse, estimate shipping, and sync carts. However, **checkout and payment require explicit human confirmation**. Agents must never finalize payment autonomously.
2. **Payment Security:** Agents must **never** capture, store, or transmit credit card details or CVVs. All transactions are securely completed on `https://shop.alshhbaa.com/checkout`.
3. **No Hallucinations:** Product prices, availability, and delivery fees must be fetched from the API.
4. **Data Privacy:** Order tokens and customer information must remain private. Administrative endpoints (`/admin`) are strictly forbidden.

For full rules and agent instructions, see [AGENTS.md](AGENTS.md).

---

## 🌐 Public Discovery Endpoints

- **Storefront:** [shop.alshhbaa.com](https://shop.alshhbaa.com)
- **OpenAPI 3.1:** [shop.alshhbaa.com/openapi.json](https://shop.alshhbaa.com/openapi.json)
- **MCP Server Card:** [shop.alshhbaa.com/.well-known/mcp/server-card.json](https://shop.alshhbaa.com/.well-known/mcp/server-card.json)
- **Agent Skills Index:** [shop.alshhbaa.com/.well-known/agent-skills/index.json](https://shop.alshhbaa.com/.well-known/agent-skills/index.json)
- **Agent Permissions:** [shop.alshhbaa.com/.well-known/agent-permissions.json](https://shop.alshhbaa.com/.well-known/agent-permissions.json)
- **LLMs Briefing:** [shop.alshhbaa.com/llms.txt](https://shop.alshhbaa.com/llms.txt) & [shop.alshhbaa.com/agenticweb.md](https://shop.alshhbaa.com/agenticweb.md)
- **Developer Portal:** [shop.alshhbaa.com/developers](https://shop.alshhbaa.com/developers)

---

## 📄 License

This repository is licensed under the [MIT License](LICENSE).
Copyright (c) 2026 Alshhbaa Copperware (متجر نحاسيات الشهباء).
