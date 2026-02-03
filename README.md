# Polymarket Arbitrage Bot

Polymarket **arbitrage bot** for 15-minute Up/Down markets. Automates the **dump-and-hedge** strategy with configurable thresholds, stop-loss hedging, and optional simulation mode. Full credential management, CLOB order execution, and market discovery via Gamma API.

[![Node.js](https://img.shields.io/badge/Node.js-16+-green.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5+-blue.svg)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-yellow.svg)](LICENSE)

## 🎯 Overview

This **Polymarket arbitrage bot** runs a **dump-and-hedge** strategy on Polymarket’s 15m Up/Down markets (e.g. BTC, ETH, SOL, XRP) by:

- **Market discovery** – Finds the current 15m market per asset via Gamma API slug
- **Price monitoring** – Polls CLOB orderbooks for Up/Down bid/ask and time remaining
- **Dump detection** – In the first N minutes of each period, detects a sharp price drop on one side (Up or Down)
- **Leg 1** – Buys the dumped side at the dip
- **Hedge (Leg 2)** – Waits until combined cost (leg1 + opposite ask) is at or below target (e.g. ≤ 0.95), then buys the opposite side to lock in profit
- **Stop-loss hedge** – If the hedge condition isn’t met within a max wait time, hedges anyway to limit risk
- **Settlement** – On market close, redeems winning outcome tokens and tracks P&L
- **Simulation mode** – Run without placing real orders (default); switch to production when ready
- **Type-safe** – Full TypeScript with strict types and clear config via `.env`

Perfect for automating the dump-and-hedge arbitrage on Polymarket 15m markets with controllable risk and optional dry-run.

<!-- Add screenshots/demo images here -->
<!--
![Bot Dashboard](docs/images/dashboard.png)
![Trade Execution](docs/images/trades.png)
-->

## ✨ Key Features

### 🚀 Trading & Strategy
- **Dump-and-hedge** – Buy the dip on one outcome, then hedge when sum of prices ≤ target
- **Multi-asset** – Supports BTC, ETH, SOL, XRP 15m markets (configurable via `MARKETS`)
- **Automatic market discovery** – Resolves current 15m market by slug and period timestamp
- **Period rollover** – Detects new 15m periods and switches to the new market automatically
- **Stop-loss hedge** – Time-based fallback hedge if ideal hedge price isn’t reached

### 🛡️ Risk & Safety
- **Simulation by default** – No real orders until you set `PRODUCTION=true` or use `npm run prod`
- **Configurable sizing** – Shares per leg, sum target, move threshold, and watch window
- **Stop-loss parameters** – Max wait before forced hedge and stop-loss percentage
- **Position tracking** – Per-period and total P&L; redemption of winning tokens on close

### 🔧 Production-Ready
- **Env-based config** – All settings in `.env` (no config files to commit)
- **CLOB auth** – API key derivation from signer or optional explicit API key/secret/passphrase
- **Proxy wallet support** – Optional Polymarket proxy/profile address and signature type (EOA / Proxy / GnosisSafe)
- **History logging** – Append-only `history.toml` for audit and debugging
- **Graceful handling** – Continues monitoring on transient API errors; clear stderr logging

## 🚀 Quick Start

### Prerequisites

- **Node.js 16+** – [Download Node.js](https://nodejs.org/)
- **Polygon wallet** – With USDC for trading (production)
- **POL/MATIC** – For gas when redeeming winning tokens (production)

### Installation

```bash
# Clone the repository
git clone https://github.com/Magic-Academy/polymarket-arbitrage-bot.git
cd polymarket-arbitrage-bot

# Install dependencies
npm install

# Build the project
npm run build
```

### Configuration

1. **Create environment file:**
```bash
cp .env.example .env
```

2. **Edit `.env` with your settings:**
```env
# Required for production (real trades and redemption)
PRIVATE_KEY=0x...                    # Your wallet private key (hex, with or without 0x)
PROXY_WALLET_ADDRESS=0x...           # Polymarket proxy/profile address (if using proxy)
SIGNATURE_TYPE=2                      # 0=EOA, 1=Proxy, 2=GnosisSafe

# Markets: comma-separated (btc, eth, sol, xrp)
MARKETS=btc

# Strategy (defaults are fine to start)
DUMP_HEDGE_SHARES=10
DUMP_HEDGE_SUM_TARGET=0.95
DUMP_HEDGE_MOVE_THRESHOLD=0.15
DUMP_HEDGE_WINDOW_MINUTES=2
DUMP_HEDGE_STOP_LOSS_MAX_WAIT_MINUTES=5
DUMP_HEDGE_STOP_LOSS_PERCENTAGE=0.2

# Simulation (default) vs production
PRODUCTION=false
```

3. **Run the bot:**
```bash
# Simulation (default) – no real orders
npm start
# or explicitly
npm run sim

# Production – real trades (ensure PRIVATE_KEY and optional PROXY_WALLET_ADDRESS are set)
npm run prod

# Development – run TypeScript with ts-node
npm run dev
```

Logs go to stderr and are appended to `history.toml`.

## ⚙️ Configuration Guide

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| **Polymarket API** | | |
| `GAMMA_API_URL` | Gamma API base URL | `https://gamma-api.polymarket.com` |
| `CLOB_API_URL` | CLOB API base URL | `https://clob.polymarket.com` |
| `API_KEY` | CLOB API key (optional; derived from signer if not set) | - |
| `API_SECRET` | CLOB API secret | - |
| `API_PASSPHRASE` | CLOB API passphrase | - |
| **Wallet & auth** | | |
| `PRIVATE_KEY` | Wallet private key (required for trading/redemption) | - |
| `PROXY_WALLET_ADDRESS` | Polymarket proxy wallet address | - |
| `SIGNATURE_TYPE` | `0` EOA, `1` Proxy, `2` GnosisSafe | `2` |
| **Polling** | | |
| `CHECK_INTERVAL_MS` | Orderbook poll interval (ms) | `1000` |
| `MARKET_CLOSURE_CHECK_INTERVAL_SECONDS` | How often to check market closure | `20` |
| **Markets & strategy** | | |
| `MARKETS` | Comma-separated: `btc`, `eth`, `sol`, `xrp` | `btc` |
| `DUMP_HEDGE_SHARES` | Shares per leg | `10` |
| `DUMP_HEDGE_SUM_TARGET` | Target sum for hedge (e.g. 0.95) | `0.95` |
| `DUMP_HEDGE_MOVE_THRESHOLD` | Dump detection threshold (e.g. 0.15 = 15%) | `0.15` |
| `DUMP_HEDGE_WINDOW_MINUTES` | Watch window for dump (minutes) | `2` |
| `DUMP_HEDGE_STOP_LOSS_MAX_WAIT_MINUTES` | Max wait before stop-loss hedge | `5` |
| `DUMP_HEDGE_STOP_LOSS_PERCENTAGE` | Stop-loss percentage | `0.2` |
| **Mode** | | |
| `PRODUCTION` | `true` = real trades, `false` = simulation | `false` |

### Optional API overrides

If you don’t set `API_KEY` / `API_SECRET` / `API_PASSPHRASE`, the bot derives CLOB credentials from the signer (recommended). You can set them explicitly if you already have API keys.

## 📖 How It Works

### Dump-and-hedge flow

1. **Discovery** – For each asset in `MARKETS`, the bot finds the current 15m Up/Down market via Gamma API (slug pattern e.g. `btc-updown-15m-<timestamp>`).
2. **Monitoring** – Every `CHECK_INTERVAL_MS`, it fetches Up/Down orderbooks, computes best bid/ask, and builds a snapshot (prices + time remaining in the period).
3. **Watch window** – For the first `DUMP_HEDGE_WINDOW_MINUTES` of each 15m period, it watches for a **dump**: one side’s ask drops by at least `DUMP_HEDGE_MOVE_THRESHOLD` (e.g. 15%) within a short time window.
4. **Leg 1** – When a dump is detected, it buys `DUMP_HEDGE_SHARES` of that side (Up or Down) at the current ask.
5. **Hedge condition** – It then waits until `leg1_entry_price + opposite_ask ≤ DUMP_HEDGE_SUM_TARGET` (e.g. ≤ 0.95).
6. **Leg 2** – When the condition is met, it buys the same number of shares of the opposite outcome (hedge). Combined cost per “share pair” is ≤ target, so resolution pays $1 per share and locks in profit.
7. **Stop-loss** – If the hedge condition isn’t met within `DUMP_HEDGE_STOP_LOSS_MAX_WAIT_MINUTES`, it executes the hedge anyway at the current price (stop-loss hedge).
8. **New period** – When the 15m period rolls over, the bot discovers the new market and resets strategy state for the new period.
9. **Closure** – After market end, it checks resolution, redeems winning outcome tokens (production only), and updates P&L. Closure checks run every `MARKET_CLOSURE_CHECK_INTERVAL_SECONDS`.

### Simulation vs production

- **Simulation** (`PRODUCTION=false` or `npm run sim`): no orders sent to the CLOB; strategy logic and logging run as normal. Use this to verify behavior and parameters.
- **Production** (`PRODUCTION=true` or `npm run prod`): real orders and redemptions. Requires `PRIVATE_KEY`; set `PROXY_WALLET_ADDRESS` and `SIGNATURE_TYPE` if you use a proxy/GnosisSafe.

## 📦 Available Scripts

| Command | Description |
|---------|-------------|
| `npm run build` | Compile TypeScript to `dist/` |
| `npm start` | Run compiled bot (simulation by default) |
| `npm run sim` | Run in simulation mode explicitly |
| `npm run prod` | Run in production (real trades) |
| `npm run dev` | Run with ts-node (development) |

## 🐳 Docker (optional)

If you add a `Dockerfile` later:

```bash
docker build -t polymarket-arbitrage-bot .
docker run --env-file .env -d --name polymarket-arbitrage-bot polymarket-arbitrage-bot
docker logs -f polymarket-arbitrage-bot
```

## 🛠️ Troubleshooting

### Bot doesn’t find markets
- Confirm `MARKETS` is one of `btc`, `eth`, `sol`, `xrp` (comma-separated).
- Check network access to Gamma and CLOB APIs; try default `GAMMA_API_URL` and `CLOB_API_URL` first.

### Orders fail in production
- Ensure `PRIVATE_KEY` is set and correct (hex, with or without `0x`).
- If using a proxy, set `PROXY_WALLET_ADDRESS` and `SIGNATURE_TYPE` (usually `2` for GnosisSafe).
- Verify USDC balance and that the market is still active and accepting orders.

### Redemption fails
- Ensure you have enough POL for gas on Polygon.
- Confirm the market is closed and resolved; the bot only redeems after resolution.

### No dumps detected
- Increase `DUMP_HEDGE_MOVE_THRESHOLD` (e.g. 0.10 → 0.15) or extend `DUMP_HEDGE_WINDOW_MINUTES`.
- Markets may be quiet; the strategy only acts when a sufficient short-term drop occurs.

## 🔐 Security Best Practices

- **Never commit `.env`** – Keep it in `.gitignore` (already listed).
- **Use env vars for secrets** – Don’t hardcode `PRIVATE_KEY` or API credentials.
- **Test in simulation first** – Run with `PRODUCTION=false` or `npm run sim` before enabling real trades.
- **Limit wallet use** – Prefer a dedicated wallet with limited funds for the bot.
- **Rotate keys** – Replace credentials if they may have been exposed.

## 📚 Project structure

- `src/main.ts` – Entry point, config load, market discovery, and monitor/trader wiring.
- `src/config.ts` – Loads and validates `.env` into typed config.
- `src/api.ts` – Polymarket Gamma + CLOB API client (markets, orderbook, orders, redemption).
- `src/monitor.ts` – Fetches orderbook snapshots and drives the strategy callback.
- `src/dumpHedgeTrader.ts` – Dump detection, leg 1/2, stop-loss hedge, closure and P&L.
- `src/models.ts` – Shared types (Market, OrderBook, TokenPrice, etc.).
- `src/logger.ts` – History log and stderr output.
- `history.toml` – Append-only log (created at runtime; in `.gitignore`).

## 🤝 Contributing

Contributions are welcome. Please open an issue or pull request.

1. Fork the repository  
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)  
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)  
4. Push to the branch (`git push origin feature/AmazingFeature`)  
5. Open a Pull Request  

## 📄 License

This project is licensed under the Apache License 2.0 – see the [LICENSE](LICENSE) file for details.

## ⚠️ Disclaimer

**IMPORTANT LEGAL DISCLAIMER:**

This software is provided “as-is” for educational and research purposes only. Trading on prediction markets involves substantial risk of loss.

- **No warranty** – The software is provided without any warranties.  
- **Use at your own risk** – You are solely responsible for any losses incurred.  
- **Not financial advice** – This is not investment or trading advice.  
- **Compliance** – Ensure compliance with local laws and regulations.  
- **Testing** – Always test with simulation and small amounts before production.

The authors and contributors are not responsible for any financial losses, damages, or legal issues arising from the use of this software.

## 📞 Support & Contact

- **Telegram**: [Telegram](https://t.me/tova34)
- **Issues**: [GitHub Issues](https://github.com/Magic-Academy/polymarket-arbitrage-bot/issues)  
- **Discussions**: [GitHub Discussions](https://github.com/Magic-Academy/polymarket-arbitrage-bot/discussions)  

## 🌟 Star history

If you find this project useful, please consider giving it a star ⭐

## 📈 Roadmap

- [ ] Optional WebSocket orderbook updates for lower latency  
- [ ] Backtesting / replay mode for strategy tuning  
- [ ] Optional Telegram/Discord notifications  
- [ ] More timeframe support (e.g. 1h)  
- [ ] PnL export and simple reporting  

---

**Keywords**: Polymarket bot, Polymarket arbitrage bot, dump and hedge, 15m Up Down, prediction markets bot, Polygon, trading automation, Polymarket CLOB
