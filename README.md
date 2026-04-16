# Sol-Intel

Solana smart money intelligence. Tracks wallets, scores tokens, sends Telegram alerts when interesting things happen on-chain.

I got tired of finding out about moves after the chart already ran. This watches the wallets that matter and tells me what they're buying before Twitter does.

## What It Does

- **Wallet monitoring** — tracks a list of wallets in real-time via Helius RPC websockets. When a tracked wallet makes a swap, you know within seconds.
- **Token safety scoring** — every token gets a score from 0-100 based on six factors before you see the alert. No more clicking into obvious rugs.
- **Telegram alerts** — structured alerts with token info, safety score, wallet label, and direct links to DexScreener/Birdeye.

## Safety Score

Each token is scored on six factors (0-100 total):

| Factor | Weight | What It Checks |
|--------|--------|----------------|
| Liquidity | 25 | Total LP value across pools |
| Holder concentration | 20 | Top 10 wallet % of supply |
| Token age | 15 | Time since mint |
| LP burn/lock | 20 | Whether LP tokens are burned or locked |
| Verification | 10 | Metadata, socials, known deployer |
| Volume | 10 | 24h trading volume vs liquidity ratio |

Scores below 40 get a ⚠️ warning. Below 20 gets a 🚫. You can set your own thresholds.

## Stack

- Python 3.11+ with `asyncio`
- [Helius](https://helius.dev/) RPC + websockets for on-chain data
- [DexScreener API](https://docs.dexscreener.com/) for market data
- [Jupiter API](https://station.jup.ag/docs/) for token metadata and routing info
- `python-telegram-bot` for alerts

## Setup

```bash
git clone https://github.com/spectarsworld/sol-intel.git
cd sol-intel
pip install -r requirements.txt
cp .env.example .env
```

Fill in `.env`:

```env
HELIUS_API_KEY=your_key
HELIUS_WS_URL=wss://atlas-mainnet.helius-rpc.com/?api-key=your_key
TELEGRAM_BOT_TOKEN=your_bot_token
TELEGRAM_CHAT_ID=your_chat_id
DEXSCREENER_API=https://api.dexscreener.com/latest
```

## Usage

```bash
# Start monitoring
python main.py

# Add a wallet to track
python manage.py add-wallet <address> --label "smart money 1"

# List tracked wallets
python manage.py list-wallets

# Test alert delivery
python manage.py test-alert
```

## Project Structure

```
sol-intel/
├── src/
│   ├── monitor/
│   │   ├── websocket.py       # Helius websocket connection
│   │   └── parser.py          # Transaction parsing & swap detection
│   ├── scoring/
│   │   ├── safety.py          # Token safety score calculator
│   │   ├── liquidity.py
│   │   ├── holders.py
│   │   └── metadata.py
│   ├── alerts/
│   │   └── telegram.py        # Telegram bot integration
│   ├── data/
│   │   ├── wallets.json       # Tracked wallet list
│   │   └── token_cache.py     # Score caching (TTL-based)
│   └── config.py
├── main.py
├── manage.py
├── requirements.txt
└── README.md
```

## Example Alert

```
🟢 Smart Money Buy

Wallet: smart money 1 (7xKX...3nFd)
Token: $BONK (DezX...kFmY)
Amount: 2.4 SOL (~$380)
Safety: 78/100 ✅

📊 DexScreener | 🪐 Birdeye
```

## Roadmap

- [ ] Portfolio tracking per wallet
- [ ] Historical performance stats for tracked wallets
- [ ] Discord bot integration
- [ ] Web dashboard
- [ ] Multi-wallet correlation (when 3+ wallets buy the same token)

## License

MIT

---

[@spectarsworld](https://github.com/spectarsworld)
