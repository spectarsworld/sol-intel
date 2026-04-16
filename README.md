# Sol-Intel

Real-time smart money tracking and token safety analysis on Solana. Monitors whale wallets, scores token risk, and pushes alerts to Telegram when interesting moves happen.

Built because I got tired of finding out about whale trades 30 minutes after everyone else.

## Features

### Wallet Tracking
- Monitor a configurable list of whale/smart money wallets
- Detect swaps, transfers, and liquidity events in real-time
- Track PnL and position changes per wallet
- Tag wallets by category (MEV bot, fund, influencer, deployer)

### Token Safety Scoring

Every token gets a safety score from 0–100 based on six factors:

| Factor | Weight | What It Checks |
|--------|--------|----------------|
| Liquidity depth | 25% | Total LP value, locked vs unlocked |
| Holder distribution | 20% | Top 10 concentration, unique holders |
| Contract risk | 20% | Mint authority, freeze authority, proxy patterns |
| Trading activity | 15% | Volume consistency, wash trading signals |
| LP lock status | 10% | Lock duration, platform (Raydium/Orca) |
| Metadata quality | 10% | Socials present, website, description accuracy |

Scores update every 60 seconds for tracked tokens.

### Telegram Alerts

```
🐋 WHALE ALERT

Wallet: Smart Money #12 (7xKp...3nRq)
Action: BUY
Token: $WIF
Amount: 142,000 USDC → 2.1M WIF
Safety Score: 74/100
DexScreener: https://dexscreener.com/solana/...

Pool: Raydium | Liquidity: $3.2M | 24h Vol: $18M
```

Alert filters: minimum trade size, safety score threshold, specific wallets only, token allowlist/blocklist.

## Architecture

```
┌──────────────────────────────────────┐
│          Helius WebSocket            │
│     (transaction streaming)          │
└──────────────┬───────────────────────┘
               │
    ┌──────────▼──────────┐
    │   Transaction Parser │
    │  (swap detection,    │
    │   amount extraction) │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐     ┌─────────────────┐
    │   Wallet Tracker     │────▶│  Telegram Bot    │
    │  (position tracking, │     │  (alerts, cmds)  │
    │   PnL calculation)   │     └─────────────────┘
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │   Token Analyzer     │
    │  ┌────────────────┐  │
    │  │ DexScreener API│  │
    │  │ Jupiter API    │  │
    │  │ Helius DAS API │  │
    │  │ On-chain reads │  │
    │  └────────────────┘  │
    └──────────────────────┘
```

All components run as asyncio tasks in a single process. No message queues, no microservices — just Python doing its thing.

## Setup

### Requirements

- Python 3.11+
- [Helius API key](https://helius.dev) (free tier works for low-volume tracking)
- Telegram bot token (via @BotFather)

### Installation

```bash
git clone https://github.com/hybridnand/sol-intel.git
cd sol-intel
pip install -e .
cp config.example.toml config.toml
```

### Configuration

Edit `config.toml`:

```toml
[helius]
api_key = "your-helius-key"
ws_url = "wss://atlas-mainnet.helius-rpc.com/?api-key=YOUR_KEY"

[telegram]
bot_token = "your-bot-token"
chat_id = "your-chat-id"
min_trade_usd = 10000        # minimum trade size for alerts
min_safety_score = 0          # alert on any score (set higher to filter)

[tracking]
poll_interval = 5             # seconds between wallet checks
score_refresh = 60            # seconds between safety score updates

[[wallets]]
address = "7xKp...3nRq"
label = "Smart Money #1"
category = "fund"

[[wallets]]
address = "4mBz...9wYt"
label = "MEV Bot Alpha"
category = "mev"
```

### Running

```bash
# Start the tracker
sol-intel run

# One-off token analysis
sol-intel score <token_mint_address>

# Add a wallet to track
sol-intel add-wallet <address> --label "New Whale" --category fund
```

## Project Structure

```
sol-intel/
├── src/
│   ├── tracker/
│   │   ├── listener.py        # Helius WebSocket consumer
│   │   ├── parser.py          # Transaction parsing (Jupiter, Raydium, Orca)
│   │   └── positions.py       # Wallet position and PnL tracking
│   ├── scoring/
│   │   ├── analyzer.py        # Token safety scoring engine
│   │   ├── liquidity.py       # LP depth and lock analysis
│   │   ├── holders.py         # Holder distribution checks
│   │   └── contract.py        # Mint/freeze authority, proxy detection
│   ├── alerts/
│   │   ├── telegram.py        # Telegram bot and alert formatting
│   │   └── filters.py         # Alert filtering rules
│   └── cli.py
├── tests/
├── config.example.toml
└── pyproject.toml
```

## Limitations

- Helius free tier has rate limits — heavy tracking may need a paid plan
- Safety scores are heuristic, not guarantees. A score of 80 doesn't mean a token is safe, just that it passes basic checks
- Jupiter/Raydium swap parsing covers the major DEXes but newer protocols may not be detected yet
- PnL tracking assumes single-entry positions (no averaging across multiple buys yet)

## License

MIT
