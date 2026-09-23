# IDX Market MCP (`idx-market-mcp`)

[![Python](https://img.shields.io/badge/python-3.11%20%7C%203.12-blue)](https://www.python.org/)
[![FastMCP](https://img.shields.io/badge/FastMCP-0.4.0+-green)](https://github.com/jlowin/fastmcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Market](https://img.shields.io/badge/Market-IDX%20%2F%20BEI-red)](https://www.idx.co.id/)

**Institutional-grade Model Context Protocol (MCP) server for Indonesian Stock Exchange (IDX) equities.**

`idx-market-mcp` bridges AI agents (Claude Desktop, Cursor, Hermes Agent) to the Indonesian capital markets. It combines public market telemetry, real-time Level 2 orderbook depth, institutional bandarmology (broker accumulation/distribution), automated chart pattern recognition, and quantitative risk budgeting into a **single, token-efficient Python server**.

---

## 🌟 Key Capabilities

1. **Unified Multi-Action Gateway (`idx_stockbit`)**:
   - Access orderbook depth, broker summaries, audited financials, corporate action dividends, and insider disclosures through a **single tool schema** (~150 input tokens), eliminating tool-definition bloat across conversation turns.
2. **Institutional Bandarmology**:
   - Real-time foreign net flows and broker summary breakdown (Top-1, Top-3, Top-5 broker accumulation/distribution status).
3. **Full Level 2 Orderbook Depth**:
   - 10 Bid & 10 Ask depth ladder with lot volume and institutional support/resistance wall detection.
4. **Algorithmic Chart Pattern Recognition**:
   - Mathematical detection of local extrema (`scipy.signal.argrelextrema`) and trendline regression to identify triangles, channels, and coils, complete with Bulkowski breakout probabilities and auto-generated candlestick charts (`mplfinance`).
5. **Defensive Financial Sanitization**:
   - Automatic currency mismatch normalization for USD-reporting issuers (`KRAS`, `AMMN`, `MEDC`, `ADRO`), thin-equity ROE clamping, and solvency violation locks ($\text{DER} > 1.5\times$).
6. **Quantitative Risk Budgeting & Two-Tranche Orders**:
   - Anti-cluster sector concentration ceilings (max 50% per sector) and mathematical 2-tranche limit order calculators ready for retail broker execution (e.g. Ajaib, Stockbit).

---

## 🏗️ Architecture

```
┌────────────────────────────────────────────────────────┐
│                   AI Agent / Client                    │
│      (Claude Desktop, Cursor, Hermes Agent, LLM)       │
└───────────────────────────┬────────────────────────────┘
                            │ Model Context Protocol (JSON-RPC / stdio)
┌───────────────────────────▼────────────────────────────┐
│                    idx-market-mcp                      │
│                                                        │
│  ┌───────────────────────┐   ┌───────────────────────┐ │
│  │ Unified Gateway       │   │ Quantitative Guards   │ │
│  │ (idx_stockbit)        │   │ (Sanitize/Sector/Lot) │ │
│  └───────────┬───────────┘   └───────────┬───────────┘ │
│              │                           │             │
│  ┌───────────▼───────────┐   ┌───────────▼───────────┐ │
│  │ Pattern Recognition   │   │ Free Quote Fallback   │ │
│  │ (scipy + mplfinance)  │   │ (Yahoo Finance .JK)   │ │
│  └───────────────────────┘   └───────────────────────┘ │
└──────────────┬───────────────────────────┬─────────────┘
               │ Session Bearer Token      │ Public OHLCV
┌──────────────▼───────────┐   ┌───────────▼─────────────┐
│    Stockbit Exodus API   │   │   Yahoo Finance Charts  │
│ (Orderbook L2 / Bandar)  │   │      (.JK Tickers)      │
└──────────────────────────┘   └─────────────────────────┘
```

---

## 🛠️ Tool Catalog (11 Active Tools)

| Tool Name | Type | Description |
|:---|:---|:---|
| `idx_stockbit` | **Gateway** | Single unified action tool: `orderbook`, `bandarmology`, `keystats`, `dividends`, `insider`. |
| `idx_get_orderbook` | Market L2 | 10-level Bid & Ask ladder, total queue lots, ARA/ARB thresholds. |
| `idx_get_bandarmology` | Flow | Broker summary, institutional flow in IDR, Top 1/3/5 broker actions. |
| `idx_get_keystats` | Fundamental | Audited IDR balance sheet, Piotroski F-Score (1-9), DER, ROE, CFO, DPR. |
| `idx_get_dividends` | Corp Action | Historical and upcoming dividend calendar (cum, ex, payment dates, IDR value). |
| `idx_get_insider_activity` | Corporate | Major shareholder and director/commissioner transaction disclosures. |
| `idx_analyze_chart_pattern` | Technical | Algorithmic pattern detection, regression slopes, and annotated PNG chart rendering. |
| `idx_get_market_quote` | Price Feed | Real-time quote, intraday range, volume in shares and lots. |
| `idx_sanitize_financials` | Risk Guard | Normalizes USD reporting mismatches, thin-equity ROE distortions, and leverage traps. |
| `idx_check_sector_concentration` | Risk Guard | Enforces 50% single-sector portfolio ceiling (e.g. anti-cluster bank guard). |
| `idx_calc_tranche_orders` | Execution | Computes disciplined 2-tranche limit orders (Tranche 1 immediate + Tranche 2 reserve). |

---

## 🚀 Quickstart

### 1. Installation

Clone and install dependencies via `pip` or `uv`:

```bash
git clone https://github.com/mpandudc/idx-market-mcp.git
cd idx-market-mcp

# Using uv (recommended)
uv venv
source .venv/bin/activate
uv pip install -e .

# Or using standard pip
pip install -e .
```

### 2. Authentication (Optional for Deep Data)

Basic market quotes, sector concentration checks, and chart pattern recognition work **without credentials** (using public feeds). 

To unlock Level 2 Orderbook depth, Bandarmology, and audited Keystats, provide a Stockbit Bearer token via environment variable:

```bash
export STOCKBIT_BEARER_TOKEN="eyJhbGciOiJSUzI1NiIs..."
```
*(Token can be obtained once from your browser DevTools `Authorization: Bearer` header on stockbit.com).*

---

## 🔌 Client Configuration

### Claude Desktop
Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "idx-market": {
      "command": "python",
      "args": ["-m", "idx_market_data_mcp.server"],
      "env": {
        "STOCKBIT_BEARER_TOKEN": "YOUR_BEARER_TOKEN_HERE"
      }
    }
  }
}
```

### Cursor
Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "idx-market": {
      "command": "python",
      "args": ["-m", "idx_market_data_mcp.server"],
      "env": {
        "STOCKBIT_BEARER_TOKEN": "YOUR_BEARER_TOKEN_HERE"
      }
    }
  }
}
```

### Hermes Agent (`~/.hermes/config.yaml`)
```yaml
mcp_servers:
  idx_market:
    command: /path/to/idx-market-mcp/.venv/bin/python
    args:
      - -m
      - idx_market_data_mcp.server
    enabled: true
    env:
      STOCKBIT_BEARER_TOKEN: "YOUR_BEARER_TOKEN_HERE"
```

---

## 📈 Example Prompts

- *"Cek orderbook dan bandarmology saham MYOR hari ini"*
- *"Analisis chart pattern SMSM pada timeframe 1d dan render gambarnya"*
- *"Hitung tiket limit order 2-tranche untuk alokasi Rp 12.000.000 di saham BBCA"*
- *"Audit portofolio saya: apakah boleh menambah BBNI jika sudah memegang BBCA, BMRI, dan BBRI?"*

---

## ⚖️ Disclaimer

This software is developed strictly for **educational, academic, and quantitative research purposes**. It is not affiliated with, endorsed by, or sponsored by PT Bursa Efek Indonesia (IDX) or PT Stockbit Sekuritas Digital. Automated data access may be subject to the terms of service of upstream providers. Use responsibly on your own personal accounts.

## 📄 License

MIT License — Copyright (c) 2026 [Muhammad Pandu Dwi Cahyo](https://github.com/mpandudc).
