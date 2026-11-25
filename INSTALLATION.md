# Quantura v2.5.0 Installation and Deployment Guide

This manual provides step-by-step instructions to install, configure, secure, and deploy **Quantura™ Trading Terminal** locally or on an **Ubuntu 22.04 / 24.04 LTS** or **Debian 12** VPS—covering both **Public GitHub Source Installation** and **Pre-Built Protected Release Deployment (`release-protected/`)**, alongside **Ed25519 License Server Verification**, PostgreSQL, PM2, Nginx SSL, Binance API, Telegram Dual Publishing, **12 Quantitative Strategies**, and **Smart Adaptive 2FA**.

> **Quick Links:** [Overview (README)](README.md) · [English HTML Manual](documentation/index.html) · [Arabic HTML Manual](public/Quantura_Documentation.html)

---

## Table of Contents

- [1. System Requirements](#1-system-requirements)
- [2. Local Development Setup](#2-local-development-setup)
- [3. Production VPS Setup](#3-production-vps-setup)
- [4. Nginx Reverse Proxy and SSL](#4-nginx-reverse-proxy-and-ssl)
- [5. 12 Strategies and Institutional Risk Phases](#5-12-strategies-and-institutional-risk-phases)
- [6. Telegram Dual Publishing](#6-telegram-dual-publishing)
- [7. Binance API and Execution Modes](#7-binance-api-and-execution-modes)
- [8. Private Server Access and Smart 2FA](#8-private-server-access-and-smart-2fa)
- [9. License Server Integration and Protected Builds](#9-license-server-integration-and-protected-builds)
- [10. Maintenance Testing and Troubleshooting](#10-maintenance-testing-and-troubleshooting)
- [11. Official Support and Licensing](#11-official-support-and-licensing)

---

## 1. System Requirements

- **Operating System:** Ubuntu 22.04 / 24.04 LTS, Debian 12 (Bookworm), or macOS/Windows (for local development)
- **Hardware (VPS Recommended):** 2+ vCPUs, 2 GB+ RAM, 15 GB SSD
- **Runtime:** Node.js `v20.0.0+` (`v22 LTS` recommended)
- **Database:** PostgreSQL `14+` / `16+`
- **Network Ports:** `22` (SSH), `80` (HTTP), `443` (HTTPS), internal app port `3000`

[Back to Menu](#table-of-contents)

---

## 2. Local Development Setup

```bash
# 1. Install dependencies
npm install

# 2. Copy environment template and configure secrets
cp .env.example .env

# 3. Start full-stack dev server (Express API + Vite frontend on port 3000)
npm run dev
```

Navigate to `http://localhost:3000`.

[Back to Menu](#table-of-contents)

---

## 3. Production VPS Setup

### Step 1: System Updates and Required Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git build-essential ufw unzip nginx certbot python3-certbot-nginx postgresql postgresql-contrib
```

### Step 2: Provision PostgreSQL Database
```bash
sudo -u postgres psql
```
Run inside the `psql` prompt:
```sql
CREATE DATABASE quantura;
CREATE USER quantura_user WITH ENCRYPTED PASSWORD 'your_strong_password';
GRANT ALL PRIVILEGES ON DATABASE quantura TO quantura_user;
\c quantura
GRANT ALL ON SCHEMA public TO quantura_user;
\q
```

### Step 3: Install Node.js 22 LTS and PM2
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
node -v
```

### Step 4: Download Source from Public Repo OR Extract Pre-Built Package
Choose the deployment method matching your setup:

**Method A — Clone Public GitHub Repository:**
```bash
sudo mkdir -p /var/www/quantura
sudo chown -R $USER:$USER /var/www/quantura
git clone https://github.com/your-org/quantura.git /var/www/quantura
cd /var/www/quantura
```

**Method B — Extract Pre-Built Protected Distribution (`release-protected/`):**
```bash
sudo mkdir -p /var/www/quantura
sudo chown -R $USER:$USER /var/www/quantura
cd /var/www/quantura
unzip quantura-protected-release.zip
```

### Step 5: Configure Environment Variables
```bash
cp .env.example .env
nano .env
```
Populate `.env`:
```env
PORT=3000
DATABASE_URL="postgresql://quantura_user:your_strong_password@localhost:5432/quantura"

# Authentication & Encryption (Always set strong unique secrets before first start)
SESSION_SECRET="generate_a_long_random_64_char_session_secret"
JWT_SECRET="generate_a_long_random_64_char_secret"
ENCRYPTION_KEY="generate_a_32_character_secret_key"
ADMIN_USERNAME="admin"
ADMIN_PASSWORD="set_your_strong_admin_password"
ADMIN_EMAIL="admin@yourdomain.com"

# Licensing & Ed25519 Verification
QUANTURA_LICENSE_KEY="YOUR_LICENSE_KEY"
LICENSE_SERVER_URL="https://quanturalicense.myvnc.com"
LICENSE_PUBLIC_KEY=""
STRICT_LICENSE_SIGNATURE="false"
QUANTURA_COMMERCIAL_BUILD="true"

# AI & Telegram Dual Publishing (Optional — can also be configured in Settings UI)
GEMINI_API_KEY=""
DEEPSEEK_API_KEY=""
QWEN_API_KEY=""
QWEN_BASE_URL=""
OPENAI_API_KEY=""
TELEGRAM_BOT_TOKEN=""
TELEGRAM_CHAT_ID=""
TELEGRAM_CHANNEL_ID=""

# Default Binance Setup (Optional — recommended to configure via Encrypted UI Modal)
BINANCE_API_KEY=""
BINANCE_API_SECRET=""
BINANCE_MARKET_TYPE="FUTURES"
```

### Step 6: Install Dependencies, Migrate Database, and Build

**If deploying from Public GitHub Source (Method A):**
```bash
npm install
npx prisma generate
npx prisma migrate deploy
npm run build
```

**If deploying Pre-Built Protected Package (Method B — no build step required):**
```bash
npm install --omit=dev
npx prisma generate
npx prisma migrate deploy
```

### Step 7: Launch 24/7 Background Service with PM2
```bash
pm2 start npm --name "quantura" -- start
pm2 save
pm2 startup
pm2 status
```

[Back to Menu](#table-of-contents)

---

## 4. Nginx Reverse Proxy and SSL

Create `/etc/nginx/sites-available/quantura`:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;

        # WebSocket support for Binance streams & live updates
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

Enable the site, obtain a free Let's Encrypt certificate, and enable the firewall:
```bash
sudo ln -s /etc/nginx/sites-available/quantura /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
sudo certbot --nginx -d your-domain.com

sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

[Back to Menu](#table-of-contents)

---

## 5. 12 Strategies and Institutional Risk Phases

- **30 Trading Pairs & Multi-Timeframe Strategy Specialization:** Quantura monitors 30 USDT pairs continuously. In `AUTO` timeframe mode, each active strategy scans every symbol on its specialized timeframe (`15m`, `30m`, `1h`, `4h`) and ranks valid candidates by Confidence and Risk/Reward (`>= 1:2.05`).
- **Unrestricted Selection (`0–12`):** Arm any number of strategies—1, 2, 3, or all 12—from **Smart Strategies** or **Auto Bot**. Setting `0/12` places the scanner in passive market monitoring mode with zero forced minimum strategy lock.

| # | Strategy ID | Timeframe | Lev | Min Conf | Edge Summary |
| :---: | :--- | :---: | :---: | :---: | :--- |
| 1 | `INSTITUTIONAL_SMC` | `1h` | `2x` | `75%` | Order Blocks, Fair Value Gaps & Liquidity Sweeps |
| 2 | `MOMENTUM_TREND_PRO` | `1h` | `3x` | `85%` | EMA50/200 Alignment, ADX Acceleration & Volume Surge |
| 3 | `SWING` | `4h` | `2x` | `75%` | 4H Macro Trend Swings with Lowest Portfolio Drawdown |
| 4 | `MTF_CONFLUENCE` | `15m` | `3x` | `72%` | Tri-Timeframe Agreement (`4H + 1H + 15m`) |
| 5 | `VWAP_VOLUME_DELTA` | `30m` | `3x` | `70%` | VWAP Pullbacks & Order Book Delta Imbalance |
| 6 | `FUNDING_SQUEEZE` | `1h` | `3x` | `75%` | Short/Long Squeeze Setups on Extreme Funding Bias |
| 7 | `BREAKOUT` | `30m` | `4x` | `70%` | Bollinger Squeeze Expansion & Volume Breakouts |
| 8 | `SCALPER` | `15m` | `5x` | `60%` | 15m High-Frequency Pullbacks & Rapid Trailing |
| 9 | `MEAN_REVERSION` | `15m` | `3x` | `68%` | Rebounds at RSI & Bollinger Extremes (`ADX < 24`) |
| 10 | `LIQUIDITY_HUNT` | `15m` | `3x` | `75%` | Stop-Hunt Sweeps & Fakeout Traps with Delta Absorption |
| 11 | `ORDER_BLOCK_DISPLACEMENT` | `1h` | `3x` | `75%` | Order-Block & FVG Mitigation after Impulsive Displacement |
| 12 | `SESSION_ORB_BREAKOUT` | `30m` | `3x` | `74%` | Institutional Range Expansion with Volume & `+DI/-DI` |

- **Entry & Profit Protection Defaults:**
  - **Anti-Chase Gate:** Blocks extended entries beyond `1.45x ATR` from EMA20 (`1.60x` Breakout, `1.85x` confirmed Liquidity Sweep reclaim).
  - **Structural Stop Loss:** Bounded post-sweep SL (`1.25x–2.35x ATR`, up to `2.8x ATR` for Swing).
  - **1R-Aligned Trailing & Early Secure:** `RoeEngine` waits for `>= 1.0R` and `>= 6.0% Net ROE` before arming `ATR_DYNAMIC` trailing; **Early Secure** locks `35%` + fee-aware breakeven once `>= 1.15R` (`>= 5.0%` Futures / `>= 1.5%` Spot Net ROE) is reached.
- **5 Institutional Execution & Quant Safeguard Phases:**
  1. **Phase 1 — Differential Sync & Relational Trade DB (`robustSyncEngine` + `tradeDb`):** Eliminates redundant DB writes during mark-to-market price ticks.
  2. **Phase 2 — Live Exchange Sync & Idempotent Order Manager (`binanceWebSocket` + `binanceUserStream` + `liveOrderManager`):** Deterministic `clientOrderId` idempotency and real-time fill sync.
  3. **Phase 3 — Smart Order Router & TCA (`smartOrderRouter`):** Intelligent maker/limit/market routing with Transaction Cost Analysis.
  4. **Phase 4 — Correlation, Flow & Event Blackout Guard (`institutionalFlow` + `RollingCorrelation` + `EventBlackoutGuard`):** Dynamic Pearson correlation exposure caps, institutional flow scoring, and macro blackout protection.
  5. **Phase 5 — Monte Carlo, Walk-Forward & Multi-Venue Hedge (`backtestEngine` + `multiVenueHedging`):** Statistical stress-testing and multi-venue hedge analytics.

[Back to Menu](#table-of-contents)

---

## 6. Telegram Dual Publishing

Quantura supports simultaneous **Private Bot** notifications and **Public Channel** broadcasts (`Quantura-bot` → `@Quantura`):

1. Create your bot via `@BotFather` and copy `TELEGRAM_BOT_TOKEN`.
2. Set `TELEGRAM_CHAT_ID` to your personal Telegram numeric chat ID for private interactive commands (`/status`, `/balance`, `/positions`, `/scanner`, `/signals`, `/start_bot`, `/stop_bot`).
3. Add the bot as an **Administrator** on your Telegram Channel (with **Post Messages** permission).
4. Set `TELEGRAM_CHANNEL_ID` (either `@ChannelUsername` or `-100xxxxxxxxxx`) in `.env` or inside **Settings → Telegram Notifications** and enable **Dual Channel Publishing**.
5. **Paper Trading Transparency Rule:** Channel broadcasts operate strictly as a read-only consumer of verified Paper Trading events (`PAPER TRADING | SIMULATED EXECUTION`), publishing accurate simulated metrics and persistent simulation disclosures without altering trading execution mode.

[Back to Menu](#table-of-contents)

---

## 7. Binance API and Execution Modes

Quantura enforces strict separation across **Market Types (`SPOT` vs `USDT-M FUTURES`)** and **Execution Modes (`PAPER`, `BINANCE_TESTNET`, `BINANCE_LIVE`)**:

1. **Paper Simulation (`PAPER`):** Default safe mode with isolated Spot & Futures paper ledgers using live market prices. No API keys required and no real-money orders executed.
2. **Binance Testnet (`BINANCE_TESTNET`):**
   - Spot Testnet keys: `https://testnet.binance.vision`
   - Futures Testnet keys: `https://testnet.binancefuture.com`
3. **Binance Live (`BINANCE_LIVE`):**
   - Create an API Key in Binance **API Management**.
   - Enable **Enable Spot & Margin Trading** and/or **Enable Futures**.
   - **Never** enable Withdrawals.
   - Copy the VPS outbound IP displayed inside Quantura's **Binance API Setup Modal** and add it to Binance's **Restrict access to trusted IPs only** whitelist.

[Back to Menu](#table-of-contents)

---

## 8. Private Server Access and Smart 2FA

Quantura operates as a **single-owner private trading server**. Public user registration is permanently disabled in the UI and rejected (`403 Forbidden`) at `/api/auth/register`.

1. Set `SESSION_SECRET`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`, and `ADMIN_EMAIL` in `.env` **before** starting the server on a fresh installation (no default credentials are exposed in the login UI).
2. Open `https://your-domain.com` and sign in via the instant Owner Sign-In console. Failed password or 2FA attempts trigger progressive anti-brute-force cooldowns.
3. On initial bootstrap password login, rotate your owner password immediately when prompted, then complete guided **Smart 2FA** enrollment (QR scan + 6-digit confirmation) and store your **8 one-time emergency recovery codes (`QNT-XXXX-XXXX`)** safely offline.
4. Manage advanced security controls anytime in **Settings → Security & 2FA** or from the sidebar drawer:
   - **Auto-Submitting 6-Box OTP:** Server-authoritative RFC-6238 TOTP validation synced with server time.
   - **Adaptive Device Trust (30 Days) vs. Strict 2FA Mode:** Optionally trust a workstation for 30 days, or toggle **Strict 2FA Always** to require verification on every sign-in and ignore trusted device tokens.
   - **Trusted Devices & Recovery Management:** View active trusted devices, revoke individual browsers or revoke all devices, regenerate 8 fresh recovery codes, or configure a PBKDF2-hashed 6-digit fallback **Master PIN**.

[Back to Menu](#table-of-contents)

---

## 9. License Server Integration and Protected Builds

Quantura is distributed under a commercial licensing model where users purchase a **License Key** validated against your central **License Server** (`LICENSE_SERVER_URL`).

### How End-User Activation Works
1. On startup, Quantura checks `QUANTURA_LICENSE_KEY` from `.env`, stored KV activation state, or `.quantura_certificate.json`.
2. When `QUANTURA_COMMERCIAL_BUILD=true` (automatic inside `release-protected/`), **zero default or trial keys exist**—the app starts as `UNLICENSED` until activated against `LICENSE_SERVER_URL`.
3. Every activation validates the key alongside the server's hardware fingerprint (`HWID`) and seals `.quantura_certificate.json` with a hardware-bound seal (`72h` maximum outage grace window). Explicit license denials (`INVALID`, `REVOKED`, `EXPIRED`, `HWID_MISMATCH`) block execution immediately without offline bypass.

### License Publisher Workflow (For Repo Owner)
1. **Generate Ed25519 Keypair:**
   ```bash
   npm run license:keygen
   ```
   - Keep `keys/license_private.pem` **strictly on your License Server** (never commit or ship it; `keys/` is gitignored).
   - Use the printed Node.js signing snippet in `POST /api/v1/license/verify` on your License Server to attach `signedPayload` + Ed25519 `signature`.
2. **Configure Public Key for Public Repo Deployments:**
   - Provide the single-line PEM string via `LICENSE_PUBLIC_KEY` in `.env` (or embed it into compiled releases).
3. **Optional — Generate Source-Free Protected Package (`release-protected/`):**
   ```bash
   # Standard adaptive protected build (verifies signed responses if present & seals local certs):
   npm run build:dist-protected

   # Strict Ed25519 build (rejects any license response missing a valid Ed25519 signature):
   npm run build:dist-protected -- --strict-signature
   ```
   The output folder `release-protected/` contains compiled frontend assets, obfuscated `dist/server.cjs`, `prisma/`, documentation, and a minimal production `package.json` with zero `.ts`/`.tsx` source files or fallback keys.

[Back to Menu](#table-of-contents)

---

## 10. Maintenance Testing and Troubleshooting

| Task | Command |
| :--- | :--- |
| **Inspect Live Server Logs** | `pm2 logs quantura` |
| **Restart Application** | `pm2 restart quantura` |
| **Type-Check Codebase** | `npm run lint` |
| **Run Core Test Suites** | `npm test && npx tsx --test tests/licenseSecurity.test.ts src/utils/privateAuth2FA.test.ts src/utils/institutionalUpgrades.test.ts src/utils/licenseIntelligence.test.ts` |
| **Run Quant Engine Assertions** | `npx tsx tests/riskEngine.test.ts && npx tsx tests/entryEngine.test.ts && npx tsx src/server/roeEngine/roeEngine.test.ts` |
| **Verify Health & Scanners** | `curl -s http://localhost:3000/api/health && curl -s http://localhost:3000/api/bot/status` |
| **Verify Database Health** | `curl -s http://localhost:3000/api/db/status` |

### Common Checks
- **`502 Bad Gateway` in Nginx:** Confirm `pm2 status` shows `online` and `.env` has `PORT=3000`.
- **`UNLICENSED` or `INVALID_LICENSE_SIGNATURE`:** Confirm `QUANTURA_LICENSE_KEY` matches an active license on `LICENSE_SERVER_URL`, ensure `LICENSE_PUBLIC_KEY` matches your server's `license_private.pem`, and delete `.quantura_certificate.json` before re-activating if migrating to a new server HWID.
- **Binance `-2015 Invalid API-key, IP, or permissions`:** Verify Spot vs. Futures permission matches your active market toggle, Testnet keys are used only in Testnet mode, and your server IP is whitelisted.
- **Prisma Migration Error:** Verify `DATABASE_URL` credentials in `.env` and ensure PostgreSQL service is running (`sudo systemctl status postgresql`).

[Back to Menu](#table-of-contents)

---

## 11. Official Support and Licensing

For installation help, VPS setup assistance, license upgrades (`PRO` / `VIP` / `ENTERPRISE`), or HWID transfers:

- **Email:** `jawman27227@gmail.com`
- **WhatsApp:** `+212660323113`
- **Telegram:** `+212660323113` (`https://t.me/+212660323113`)
