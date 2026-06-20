<div align="center">

# 🔮 TCG Oracle — WebMCP

**The world's first on-chain price oracle with native WebMCP support.**

Make 432,000+ trading card prices, graded premiums, and on-chain Merkle proofs
discoverable by any AI agent — directly from the browser.

[![WebMCP](https://img.shields.io/badge/WebMCP-Chrome_146+-4285F4?logo=googlechrome&logoColor=white)](https://developer.chrome.com/docs/ai/webmcp)
[![Oracle](https://img.shields.io/badge/Oracle-Live-00C853)](https://oracle.the-undesirables.com)
[![LitVM](https://img.shields.io/badge/LitVM-LiteForge-FF6F00)](https://liteforge.explorer.caldera.xyz)
[![License](https://img.shields.io/badge/License-BUSL--1.1-blue)](LICENSE)

</div>

---

## What is WebMCP?

[WebMCP](https://developer.chrome.com/docs/ai/webmcp) is a new web standard (co-authored by Google & Microsoft, backed by W3C) that lets websites expose **structured, callable tools** to AI agents browsing the page.

Instead of an AI agent scraping your site and guessing what buttons to click, WebMCP gives it a clean API contract — like MCP, but native to the browser.

**This module registers 7 oracle tools** that any AI agent can discover and call just by visiting a page that includes this script.

## Quick Start

Add one line to any HTML page:

```html
<script src="https://oracle.the-undesirables.com/static/tcg-oracle-webmcp.js"></script>
```

Or via jsDelivr CDN:

```html
<script src="https://cdn.jsdelivr.net/gh/sailorpepe/tcg-oracle-webmcp@latest/tcg-oracle-webmcp.js"></script>
```

That's it. Any AI agent with WebMCP support will automatically discover the tools.

> **Note:** WebMCP requires Chrome 146+ with the flag enabled:
> `chrome://flags/#enable-webmcp-testing` → **Enabled** → Relaunch.
> As of June 2026, WebMCP is in origin trials (Chrome 149).

## Available Tools

| Tool | Description | Endpoint |
|------|-------------|----------|
| `tcg_search` | Search 432K+ cards by name across 13 TCG categories | Free |
| `tcg_price` | Get current price + 60 days of daily history | Free |
| `tcg_graded_premiums` | PSA 10, PSA 9, BGS 9.5 prices from eBay solds | Free |
| `tcg_categories` | List all 13 supported game categories | Free |
| `tcg_merkle_proof` | On-chain Merkle proof (LitVM LiteForge) | Free |
| `tcg_oracle_stats` | Live oracle statistics and data freshness | Free |
| `tcg_top_movers` | Top gaining/losing cards over 7 days | Free |

## How AI Agents Use It

When an AI agent visits a page with this script loaded, it can call tools like this:

```
Agent: "What's the current price of a Charizard Base Set?"

→ Agent discovers tcg_search via navigator.modelContext
→ Calls tcg_search({ query: "Charizard Base Set" })
→ Gets product_id: 84198, market_price: $42.38
→ Calls tcg_graded_premiums({ product_id: 84198 })
→ Gets PSA 10: $285.00, PSA 9: $112.50
→ Returns structured answer to user
```

No scraping. No guessing. No API keys. The tools are structured, typed, and self-documenting.

## Example: Agent Workflow

```javascript
// An AI agent browsing your page can do this:
const tools = await navigator.modelContext.getTools();
// → Returns all 7 TCG Oracle tools with schemas

// Search for a card
const results = await tools.tcg_search.call({ query: "Black Lotus" });
// → { products: [{ id: 1196, name: "Black Lotus", market_price: 44530 }] }

// Get graded premiums
const graded = await tools.tcg_graded_premiums.call({ product_id: 1196 });
// → { grades: [{ grade: "PSA 10", price: 540000 }, ...] }

// Verify on-chain
const proof = await tools.tcg_merkle_proof.call({ product_id: 1196 });
// → { root: "0x7a3f...", proof: [...], contract: "0xc159..." }
```

## Architecture

```
┌──────────────────────────────────────────────┐
│  Browser (Chrome 146+)                       │
│  ┌────────────────────────────────────────┐  │
│  │  Web Page                              │  │
│  │  <script src="tcg-oracle-webmcp.js">   │  │
│  │                                        │  │
│  │  navigator.modelContext.addTool(...)    │  │
│  │    → tcg_search                        │  │
│  │    → tcg_price                         │  │
│  │    → tcg_graded_premiums               │  │
│  │    → tcg_categories                    │  │
│  │    → tcg_merkle_proof                  │  │
│  │    → tcg_oracle_stats                  │  │
│  │    → tcg_top_movers                    │  │
│  └──────────────┬─────────────────────────┘  │
│                 │ fetch()                     │
└─────────────────┼────────────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────┐
│  oracle.the-undesirables.com                 │
│  ┌──────────┐  ┌───────────┐  ┌───────────┐ │
│  │ 432K     │  │ 12.4M     │  │ Merkle    │ │
│  │ Products │  │ Prices    │  │ Proofs    │ │
│  └──────────┘  └───────────┘  └───────────┘ │
│                                              │
│  SQLite (WAL) → FastAPI → LitVM LiteForge   │
└──────────────────────────────────────────────┘
```

## On-Chain Verification

Every price returned by the oracle can be cryptographically verified on-chain:

| Contract | Address | Chain |
|----------|---------|-------|
| TCGPriceOracleV2 | `0x697bF6AE96fb05a47106abd012C39855A16a720E` | LitVM LiteForge (4441) |
| GradedPriceOracle | `0xc159550e9e751d6E75A0A06Bb04cfA2f59aD636B` | LitVM LiteForge (4441) |
| MerkleOracleV2 | `0x96B124f50156589274ADF8F674509374752170Cd` | LitVM LiteForge (4441) |

The `tcg_merkle_proof` tool returns a proof array compatible with OpenZeppelin's `MerkleProof.verify()`.

## Other Integration Methods

WebMCP is just one way to access the TCG Oracle. Choose the method that fits your stack:

| Method | Package | Best For |
|--------|---------|----------|
| **WebMCP** (this repo) | `<script>` tag | AI agents in browsers |
| **MCP Server** | [`pip install undesirables-mcp-server`](https://pypi.org/project/undesirables-mcp-server/) | Claude, Cursor, VS Code |
| **LitVM MCP** | [`pip install litvm-tcg-oracle`](https://pypi.org/project/litvm-tcg-oracle/) | On-chain verification |
| **ElizaOS Plugin** | [`npm i @the-undesirables/plugin-tcg-grader`](https://www.npmjs.com/package/@the-undesirables/plugin-tcg-grader) | Autonomous AI agents |
| **REST API** | [`oracle.the-undesirables.com`](https://oracle.the-undesirables.com) | Direct HTTP integration |
| **x402 Paid API** | USDC micropayments on Base | Premium data (risk forecasts, AI grading) |

## Browser Support

| Browser | Status |
|---------|--------|
| Chrome 146+ | ✅ Behind flag (`chrome://flags/#enable-webmcp-testing`) |
| Chrome 149+ | ✅ Origin trial |
| Edge | 🔄 Expected (co-authored the spec with Google) |
| Firefox | ❌ Not yet |
| Safari | ❌ Not yet |

## License

BUSL-1.1 — You can use, fork, and deploy this freely. The one restriction: you cannot launch a competing TCG price oracle service.

---

<div align="center">

**Built by [SailorPepe](https://github.com/sailorpepe) · [The Undesirables LLC](https://the-undesirables.com)**

*The world's only on-chain trading card price oracle.*

</div>
