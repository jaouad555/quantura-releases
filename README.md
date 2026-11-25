# Quantura™ v2.5.0 — Autonomous AI & Quantitative Crypto Trading Terminal

[![Version](https://img.shields.io/badge/Version-2.5.0-cyan?style=for-the-badge)](package.json)
[![Node.js](https://img.shields.io/badge/Node.js-20%2B%20%2F%2022%20LTS-emerald?style=for-the-badge)](https://nodejs.org)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%20%2B%20Prisma-blue?style=for-the-badge)](prisma/schema.prisma)
[![License](https://img.shields.io/badge/License-Commercial-orange?style=for-the-badge)](LICENSE)

**Quantura™** is a full-stack quantitative cryptocurrency trading terminal designed for **Binance Spot (1x)** and **USDT-M Perpetual Futures (1x–50x)**. It pairs isolated multi-mode execution (**Paper Simulation**, **Binance Testnet**, and **Binance Live**) across **30 monitored pairs** with **12 specialized algorithmic strategies**, stop-hunt liquidity sweep entry discipline, `1R`-aligned profit protection, institutional Risk Engine circuit breakers, **CORTEX AI** econometric calibration, forensic trade autopsy diagnostics, dual Telegram automation, and **Ed25519 license-server verification**.

---

## Navigation Menu

- [1. Core Architecture and Capabilities](#1-core-architecture-and-capabilities)
  - [Strict Market and Execution Mode Isolation](#strict-market-and-execution-mode-isolation)
  - [12 Specialized Quantitative Strategies](#12-specialized-quantitative-strategies)
  - [5 Institutional Execution and Quant Phases](#5-institutional-execution-and-quant-phases)
  - [Dual AI Intelligence and Telegram Publishing](#dual-ai-intelligence-and-telegram-publishing)
  - [Private Server Security, Smart 2FA and Ed25519 Licensing](#private-server-security-smart-2fa-and-ed25519-licensing)
- [2. Strategy Matrix](#2-strategy-matrix)
- [3. Technology Stack](#3-technology-stack)
- [4. Quick Start](#4-quick-start)
- [5. Available Scripts and Test Suites](#5-available-scripts-and-test-suites)
- [6. Documentation Files](#6-documentation-files)
- [7. Official Support and Licensing Contact](#7-official-support-and-licensing-contact)
- [8. Copyright and Disclaimer](#8-copyright-and-disclaimer)

---

## 1. Core Architecture and Capabilities

### Strict Market and Execution Mode Isolation
- **Spot vs. USDT-M Futures Isolation:** Completely separated position sizing equations, fee structures, leverage rules (`1x` Spot Long-only vs. `1x–50x` Futures Long/Short with liquidation modeling), order execution logs, and wallet ledgers.
- **Tri-Mode Environment Separation:** **Paper Trading**, **Binance Testnet**, and **Binance Live Mainnet** run with segregated active positions, closed trade histories, risk scopes, and execution logs so sandbox activity never contaminates live funds or metrics.
- **30 Monitored Trading Pairs:** Real-time Binance WebSocket price streams and cached REST klines/order-book/derivatives telemetry across 30 configured USDT pairs.

### 12 Specialized Quantitative Strategies
- **12 Specialized Strategies Across 4 Timeframes:**
  - **`15m` Execution:** `LIQUIDITY_HUNT` · `MTF_CONFLUENCE` · `MEAN_REVERSION` · `SCALPER`
  - **`30m` Intraday:** `VWAP_VOLUME_DELTA` · `BREAKOUT` · `SESSION_ORB_BREAKOUT`
  - **`1h` Institutional:** `INSTITUTIONAL_SMC` · `MOMENTUM_TREND_PRO` · `FUNDING_SQUEEZE` · `ORDER_BLOCK_DISPLACEMENT`
  - **`4h` Macro:** `SWING`
- **Unrestricted Strategy Selection (`0–12`):** Arm any number of strategies—1, 2, 3, or all 12—without artificial minimums or forced fallbacks. Setting `0/12` places the scanner in passive market monitoring mode.
- **Per-Strategy Specialized Scanning (`AUTO` Mode):** When set to `AUTO`, the Market Scanner evaluates each active strategy on its own specialized timeframe (`15m`, `30m`, `1h`, `4h`) per symbol and ranks qualified setups by **Confidence** and **Risk/Reward (`R:R`)**.
- **Liquidity Sweep & Anti-Chase Gate:** Detects 3-candle stop-hunt liquidity sweeps (`bullishSweep` / `bearishSweep`) with wick-rejection & reclaim confirmation, enforces a strict EMA20/VWAP pullback gate (`<= 1.45x ATR` standard, `<= 1.60x ATR` breakout, `<= 1.85x ATR` confirmed sweep reclaim), bounds structural Stop Losses (`1.25x–2.35x ATR`, up to `2.8x ATR` for Swing), and requires `TP1 >= 2.05 x Risk`.
- **`1.0R`-Aligned Profit Protection (`RoeEngine` & Early Secure):**
  - **Trailing Activation Guard:** `RoeEngine` waits until price reaches `>= 1.0R` AND `>= 6.0%` Net ROE before arming `ATR_DYNAMIC` trailing (`1.6x ATR`, bounded `1.0%–3.0%`).
  - **Early Secure (`35%`):** Triggers only after `>= 1.15R` AND `>= 5.0%` Futures (`>= 1.5%` Spot) Net ROE, securing `35%` and moving SL to fee-aware breakeven while leaving `65%` to capture `TP1 / TP2 / TP3`.

### 5 Institutional Execution and Quant Phases
1. **Phase 1 — Differential Sync & Relational PostgreSQL (`robustSyncEngine.ts` + `tradeDb.ts`):** Compares structural position signatures (ignoring pure mark-to-market price/ROE noise) to slash DB writes, backed by PostgreSQL relational trade persistence and Prisma models.
2. **Phase 2 — Live Exchange Sync & Idempotent Order Manager (`binanceWebSocket.ts` + `binanceUserStream.ts` + `liveOrderManager.ts`):** Real-time WebSocket order/fill reconciliation paired with deterministic `clientOrderId` idempotency.
3. **Phase 3 — Smart Order Router & TCA (`smartOrderRouter.ts`):** Routes maker/limit/market executions intelligently with live Transaction Cost Analysis tracking fees and slippage.
4. **Phase 4 — Institutional Flow, Rolling Correlation & Blackout Guard (`institutionalFlow.ts` + `RollingCorrelation.ts` + `EventBlackoutGuard.ts`):** Enforces institutional flow scoring, Pearson correlation exposure caps, and macro event blackout protection.
5. **Phase 5 — Monte Carlo, Walk-Forward & Multi-Venue Hedging (`backtestEngine.ts` + `multiVenueHedging.ts`):** Stress-tests strategies across randomized trade sequences and out-of-sample windows prior to promoting parameters to live trading, alongside **CORTEX AI** ROC-AUC / Brier / Half-Kelly calibration and the **Autopsy Room**.

### Dual AI Intelligence and Telegram Publishing
- **AI Providers:** Server-side Google Gemini (`@google/genai`) and DeepSeek / Qwen / OpenRouter reasoning in **Arabic**, **English**, and **French**.
- **Telegram Bot + Public Channel Dual Publishing:**
  - **Private Interactive Bot (`TELEGRAM_CHAT_ID`):** Real-time browser/Telegram alerts and remote commands (`/status`, `/positions`, `/balance`, `/scanner`, `/signals`, `/cortex`, `/autopsy`, `/start_bot`, `/stop_bot`).
  - **Read-Only Official Channel Publisher (`TELEGRAM_CHANNEL_ID`):** Publishes verified **Paper Trading** simulation events (`PAPER SIGNAL GENERATED`, `PAPER TRADE OPENED`, `PAPER TRADE UPDATED`, `PAPER EARLY SECURE`, `PAPER TP1/TP2/TP3`, `PAPER STOP LOSS`, `PAPER TRADE CLOSED`, and `DAILY PAPER PERFORMANCE REPORT`) clearly branded as `QUANTURA 🐺 — THE PATIENT WOLF — PAPER TRADING PERFORMANCE` with an explicit simulated-execution disclosure.

### Private Server Security, Smart 2FA and Ed25519 Licensing
- **Single-Owner Private Server Vault:** Zero-latency login console with public registration permanently disabled in both the UI and server boundary (`POST /api/auth/register` returns `403`). Backed by PBKDF2-SHA512 salted hashes, progressive brute-force cooldown locks, and HMAC-SHA256 signed `HttpOnly` cookie / in-memory Bearer sessions.
- **Server-Authoritative Smart Adaptive 2FA:** Auto-submitting 6-box RFC-6238 TOTP verification (`otpauth`), 8 salted SHA-256 one-time emergency recovery codes (`QNT-XXXX-XXXX`), PBKDF2-hashed 6-digit fallback Master PIN, and 30-day trusted device tokens (with optional **Strict 2FA Always** override and remote device revocation in **Settings → Security & 2FA**).
- **Ed25519 License Verification & Protected Distribution (`build:dist-protected`):** Supports asymmetric Ed25519 signature verification against your License Server (`LICENSE_SERVER_URL`), hardware-bound (`HWID`) sealed offline certificates with a 72-hour outage grace window, strict denial of tampered/unverified keys when `QUANTURA_COMMERCIAL_BUILD=true` or `--strict-signature` is active, and an obfuscated source-free release builder (`npm run build:dist-protected`).
- **AES-256 Encrypted API Storage:** Non-custodial key encryption with withdrawal-permission verification and outbound server IP inspection.
- **HWID License & Smart Plans Desk:** Built-in license validation (`TRIAL`, `PRO`, `VIP`, `ENTERPRISE`).

---

## 2. Strategy Matrix

| # | Canonical ID | Code | Timeframe | Ideal Futures Lev | Min Confidence | Category / Edge |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **01** | `INSTITUTIONAL_SMC` | `SMC` | `1h` | `2x` | `75%` | Order Blocks, Fair Value Gaps & Liquidity Sweeps |
| **02** | `MOMENTUM_TREND_PRO` | `MOM` | `1h` | `3x` | `85%` | EMA50/200 Alignment, ADX Acceleration & Volume Surge |
| **03** | `SWING` | `SWG` | `4h` | `2x` | `75%` | 4H Macro Trend Swings with Lowest Drawdown |
| **04** | `MTF_CONFLUENCE` | `MTF` | `15m` | `3x` | `72%` | Tri-Timeframe Agreement (4H + 1H + 15m) |
| **05** | `VWAP_VOLUME_DELTA` | `VWAP` | `30m` | `3x` | `70%` | VWAP Pullbacks & Order Book Delta Imbalance |
| **06** | `FUNDING_SQUEEZE` | `SQZ` | `1h` | `3x` | `75%` | Short/Long Squeeze Setups on Extreme Funding Bias |
| **07** | `BREAKOUT` | `BRK` | `30m` | `4x` | `70%` | Bollinger Squeeze Expansion & High-Volume Breakouts |
| **08** | `SCALPER` | `SCALP` | `15m` | `5x` | `60%` | 15m High-Frequency Pullbacks & Rapid Trailing |
| **09** | `MEAN_REVERSION` | `REV` | `15m` | `3x` | `68%` | Counter-Trend Rebounds at RSI/BB Extremes (`ADX < 24`) |
| **10** | `LIQUIDITY_HUNT` | `HUNT` | `15m` | `3x` | `75%` | Stop-Hunt Sweeps & Fakeout Traps with Absorption |
| **11** | `ORDER_BLOCK_DISPLACEMENT` | `OB-FVG` | `1h` | `3x` | `75%` | Impulsive Displacement + Order Block / FVG Mitigation |
| **12** | `SESSION_ORB_BREAKOUT` | `ORB-PRO` | `30m` | `3x` | `74%` | Institutional Range Breakout with Volume & `+DI/-DI` |

---

## 3. Technology Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | React 19, TypeScript, Tailwind CSS v4, Motion, Recharts, TradingView Lightweight Charts, Lucide Icons |
| **Backend** | Node.js 20+ (v22 LTS recommended), Express.js, TypeScript (`tsx`), `ws` WebSockets, `otpauth` |
| **Database & Persistence** | PostgreSQL 16+, Prisma ORM (`@prisma/client`), relational `tradeDb`, Write-Ahead Log (WAL), in-memory KV cache |
| **AI SDKs** | `@google/genai`, `openai` (DeepSeek / Qwen / OpenRouter compatible) |
| **Security & Build** | Node `crypto` (Ed25519, PBKDF2, HMAC-SHA256), Vite 6, `esbuild`, `javascript-obfuscator`, PM2 |

---

## 4. Quick Start

### Prerequisites
- **Node.js:** `v20.0.0+` (`v22 LTS` recommended)
- **PostgreSQL:** `v14+` / `v16+` (for production persistence)

### Option A: Public Repository Installation (Source Deployment)
```bash
git clone https://github.com/your-org/quantura.git
cd quantura
npm install
cp .env.example .env
```
Edit `.env` and configure your database, owner credentials, and license parameters:
```env
PORT=3000
DATABASE_URL="postgresql://quantura_user:your_strong_password@localhost:5432/quantura"
SESSION_SECRET="replace_with_long_random_session_secret"
JWT_SECRET="replace_with_long_random_secret"
ENCRYPTION_KEY="replace_with_32_char_secret_key"
ADMIN_USERNAME="admin"
ADMIN_PASSWORD="set_strong_admin_password"
ADMIN_EMAIL="admin@yourdomain.com"

# Licensing
QUANTURA_LICENSE_KEY="YOUR_VALID_LICENSE_KEY"
LICENSE_SERVER_URL="https://your-license-server.example.com"
LICENSE_PUBLIC_KEY=""
STRICT_LICENSE_SIGNATURE="false"
QUANTURA_COMMERCIAL_BUILD="true"
```
Run database migrations, build, and launch:
```bash
npx prisma generate
npx prisma migrate deploy
npm run build
pm2 start npm --name "quantura" -- start
pm2 save
```

### Option B: Protected Commercial Package (`release-protected/`)
To package a source-free, obfuscated release (recommended when delivering pre-built packages to license buyers):
```bash
# On maintainer machine: generate Ed25519 keys (keep keys/license_private.pem on your License Server)
npm run license:keygen
npm run build:dist-protected
# Or require signed server responses strictly:
# npm run build:dist-protected -- --strict-signature
```
On the buyer's VPS after extracting `release-protected/`:
```bash
npm install --omit=dev
cp .env.example .env
nano .env
npx prisma generate
npx prisma migrate deploy
pm2 start npm --name "quantura" -- start
pm2 save
```

*(For development mode locally, run `npm run dev` on port `3000`.)*

---

## 5. Available Scripts and Test Suites

| Script / Command | Purpose |
| :--- | :--- |
| `npm run dev` | Starts Vite + Express dev server on port `3000` via `tsx` |
| `npm run build` | Builds frontend (`dist/`) and bundles server (`dist/server.cjs`) |
| `npm run build:dist-protected` | Builds obfuscated, source-free client distribution in `release-protected/` with embedded Ed25519 key & stripped fallback keys |
| `npm run license:keygen` | Generates Ed25519 keypair in `keys/` and prints the License Server signing snippet |
| `npm start` | Runs the compiled production bundle (`node dist/server.cjs`) |
| `npm run lint` | Runs TypeScript type checking (`tsc --noEmit`) |
| `npm test` | Runs authoritative trading & reconciliation verification suite (`34/34`) |
| `npx tsx --test tests/licenseSecurity.test.ts` | Verifies Ed25519 signatures, tampering rejection, HWID binding & commercial mode enforcement |
| `npx tsx --test src/utils/privateAuth2FA.test.ts` | Tests Private Server Auth, lockout, TOTP, recovery codes, Master PIN & device trust |
| `npx tsx --test src/utils/institutionalUpgrades.test.ts` | Tests MTM differential sync, SOR/TCA, correlation, blackout guard & Monte Carlo |
| `npx tsx --test src/utils/licenseIntelligence.test.ts` | Tests License Countdown, Health Score & draft parsing |
| `npx tsx tests/riskEngine.test.ts` | Runs Risk Engine quantitative validation suite (`40` assertions) |
| `npx tsx tests/entryEngine.test.ts` | Runs Entry Quality & Anti-Chase validation suite (`25` assertions) |
| `npx tsx src/server/roeEngine/roeEngine.test.ts` | Runs `RoeEngine` profit protection & monotonicity suite (`25` assertions) |

---

## 6. Documentation Files

- **Complete VPS & Production Deployment Guide:** [INSTALLATION.md](INSTALLATION.md)
- **Interactive English Documentation (HTML):** [documentation/index.html](documentation/index.html)
- **Arabic Buyer & Deployment Manual (HTML):** [public/Quantura_Documentation.html](public/Quantura_Documentation.html)

---

## 7. Official Support and Licensing Contact

- **Email:** `jawman27227@gmail.com`
- **WhatsApp:** `+212660323113`
- **Telegram:** `+212660323113` (`https://t.me/+212660323113`)

---

## 8. Copyright and Disclaimer

Copyright © 2026 **Jaouad Abdechchafi — Quantura Systems**. All Rights Reserved.  
Quantura is a technical quantitative trading software suite and does not constitute financial or investment advice. In **Paper Trading** mode, all trades and performance figures are simulated using exchange market data without executing real-money orders. Trading Spot and leveraged USDT-M Futures involves high risk.
