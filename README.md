# shredcore-scraper-bot

A high-performance Solana trading bot written in Rust that automatically monitors social media platforms (Discord, Telegram, and X/Twitter) for trading signals and executes trades on PumpFun and PumpSwap platforms.

## Why Rust?

This bot is written in Rust, a systems programming language known for its exceptional performance, memory safety, and low latency. Rust's zero-cost abstractions and efficient execution make it ideal for high-frequency trading where every millisecond counts. The bot can process social media signals and execute trades faster than bots written in interpreted languages, giving you a competitive edge in the fast-paced world of token trading.

## Performance

This is one of the **speediest bot on the market**. With optimal server, RPC, and gRPC provider configuration, the bot can achieve **0 to 3 blocks landing speed** after signal detection. This exceptional speed ensures you're among the first to enter trades when signals are detected from social media platforms, maximizing your profit potential.

## Durable Nonce Technology

This bot uses **Durable Nonce** technology, which is essential for its operation. Here's why:

When trading at high speeds, the bot sends the same transaction to multiple SWQoS (Solana Quality of Service) providers simultaneously - including Jito, Nozomi (Temporal), and Astralane. This "spam sending" strategy dramatically improves transaction inclusion speed and success rates by ensuring your transaction reaches validators through multiple paths.

However, without durable nonces, sending the same transaction to multiple providers could result in duplicate executions if multiple providers include it in the same block. Durable nonces solve this by ensuring each transaction can only be executed once, even if it's submitted through multiple channels. This allows the bot to safely spam transactions to all available SWQoS providers for maximum speed and inclusion probability, while preventing accidental double-spends.

## Supported Platforms

- **PumpFun** - The original bonding curve platform
- **PumpSwap** - After migration from bonding curves
... Many more to come

## Features

### Social Media Scraping

- **Discord Integration**: Monitors specified Discord channels for token addresses and trading signals
  - Supports multiple guilds and channels
  - Optional author whitelist to only follow specific users
  - Self-bot authentication (requires Discord user token)

- **Telegram Integration**: Monitors Telegram channels, groups, and topics for signals
  - Supports public channels, private invite links, and topic links
  - Optional author whitelist filtering
  - Uses grammers library for reliable message streaming

- **X (Twitter) Integration**: Monitors X accounts for token mentions
  - Uses Nitter RSS feeds (no authentication required)
  - Supports multiple accounts to monitor
  - Public API access

- **Smart Signal Detection**: Automatically extracts Solana token addresses from messages with configurable confidence thresholds
- **Keyword / Username Filtering**: Include/exclude keywords / Usernames to filter relevant signals
- **Real-Time Processing**: Processes signals as they appear on social media

### Trading Features

- **Automatic Entry**: Enters trades when valid signals are detected from social media
- **Market Analysis**: Uses volume, flow, and price data to score trading opportunities
- **Dip Entry Gates**: Only enters when price is in a favorable dip range
- **Buy Flow Requirements**: Ensures sufficient buy pressure before entering
- **Interest Scoring**: Filters opportunities based on trading activity and volume

### Risk Management

- **Stop Loss**: Automatically sell if your position drops below a configured loss percentage
- **Take Profit Levels**: Set multiple profit targets with partial sell percentages
- **Dynamic Trailing Stop Loss (DTSL)**: As your profit increases, the stop loss floor automatically raises to lock in gains
- **Time-Based Exits**: Force exits after maximum hold time, negative PnL duration, or minimum profit target duration
- **Position Limits**: Control maximum concurrent positions
- **Portfolio Exposure Limits**: Limit total capital deployed across all positions

### Advanced Trading Features

- **Dollar Cost Averaging (DCA)**: Automatically add to losing positions to average down entry price
- **Re-entry Control**: Option to re-enter tokens after closing positions
- **Scalping Mode**: Automatically re-enter closed positions when conditions are favorable
- **Transaction Retries**: Automatic retry logic for failed transactions with exponential backoff
- **Blacklist**: Permanently block specific token mints from trading

### Execution Features

- **SWQoS Integration**: Simultaneously sends transactions to multiple providers (Jito, Nozomi, Astralane) for maximum inclusion speed
- **Priority Fees**: Configurable fees to encourage faster validator inclusion
- **High Slippage Tolerance**: Configured for aggressive entry/exit to ensure trades execute
- **Real-Time Market Data**: Uses gRPC (Yellowstone) or WebSocket streams for instant market updates

## Setup and Installation

### Prerequisites

- Solana CLI tools (for nonce setup)
- A Solana wallet with SOL for trading
- A license key
- RPC endpoint (high-performance private RPC recommended)
- gRPC endpoint (Yellowstone gRPC) or WebSocket endpoint
- (Optional) Discord user token for Discord scraping
- (Optional) Telegram API credentials for Telegram scraping

### Quick Start

1. **Configure the bot**:
   ```bash
   ./start.sh
   ```
   
   The interactive setup script will guide you through:
   - License key entry
   - RPC and gRPC/WebSocket URL configuration
   - Wallet private key (Base58 encoded)
   - Scraper configuration (Discord, Telegram, X)

2. **Setup Durable Nonce**:
   The setup script automatically runs `setup_nonce.sh` if no nonce account exists. This creates a durable nonce account that's required for safe multi-provider transaction sending.

3. **Configure Social Media Scrapers**:
   
   Edit `config.toml` to configure your scrapers:
   
   **Discord**:
   - Set `ENABLE_DISCORD = true`
   - Add your Discord user token to `DISCORD_USER_TOKEN`
   - Configure `DISCORD_SOURCES` with guild and channel IDs
   
   **Telegram**:
   - Set `ENABLE_TELEGRAM = true`
   - Get API credentials from https://my.telegram.org/apps
   - Add `TELEGRAM_API_ID`, `TELEGRAM_API_HASH`, and `TELEGRAM_PHONE_NUMBER`
   - Configure `TELEGRAM_SOURCES` with channel URLs
   
   **X (Twitter)**:
   - Set `ENABLE_X = true`
   - Add usernames to `X_SOURCES` array

4. **Launch the bot**:
   ```bash
   ./start.sh
   ```
   
   Or if you've already configured:
   ```bash
   ./rust-scraper
   ```

### Manual Configuration

1. Copy the example config:
   ```bash
   cp config.example.toml config.toml
   ```

2. Edit `config.toml` with your settings (see `config.example.toml` for detailed comments)

3. Setup durable nonce:
   ```bash
   ./setup_nonce.sh
   ```

4. Run the bot:
   ```bash
   ./rust-scraper
   ```

### Getting Discord Token

1. Open Discord in your browser
2. Press F12 to open Developer Tools
3. Go to Application > Local Storage > discord.com
4. Find the `token` key and copy its value
5. **Important**: Never share this token or commit it to version control

### Getting Telegram Credentials

1. Visit https://my.telegram.org/apps
2. Log in with your phone number
3. Create a new application
4. Copy the `api_id` and `api_hash`
5. Use your phone number (with country code, e.g., "+1234567890") for `TELEGRAM_PHONE_NUMBER`
6. On first run, you'll be prompted to enter a verification code sent to your phone

## Usage

### Normal Operation

Simply run `./start.sh` or `./rust-scraper` and the bot will:
1. Connect to market data streams
2. Monitor configured social media platforms for trading signals
3. Extract token addresses from messages
4. Analyze market conditions and score opportunities
5. Automatically execute buy orders when valid signals are detected
6. Manage positions with your configured risk management rules
7. Execute sells based on stop loss, take profit, or time-based rules

### Command Line Interface

You can also manually trigger trades:

**Buy a specific token**:
```bash
./rust-scraper --buy <MINT_ADDRESS> [--platform PUMP_FUN|PUMP_SWAP]
```

**Sell a position**:
```bash
./rust-scraper --sell <MINT_ADDRESS>
```

### Logs

Logs are saved to `.logs/solana-trading-bot.log` by default (can be disabled in config). Monitor this file to track bot activity, trades, scraper events, and any issues.

## Configuration Tips

### Scraper Settings

- **Confidence Threshold**: Higher `SCRAPERS_MIN_CONFIDENCE` values reduce false positives but may miss some valid mints
- **Keyword Filtering**: Use `SCRAPERS_INCLUDE_KEYWORDS` to only process messages containing specific terms
- **Author Whitelist**: Only follow signals from trusted users to reduce noise

### Trading Settings

- **Entry Filters**: Adjust `MIN_TRADES_30S`, `MIN_VOL_*` settings to filter out low-quality tokens
- **Dip Entry**: Configure `ENTRY_DIP_MIN_PCT` and `ENTRY_DIP_MAX_PCT` to control when to enter
- **Buy Flow**: Set `MIN_BUY_SHARE_FOR_ENTRY` to require stronger buy pressure

## Important Notes

- **Durable Nonce is Required**: The bot requires a durable nonce account for safe operation with multiple SWQoS providers. The setup script handles this automatically.

- **High-Performance RPC Recommended**: For best results, use a private, high-performance RPC endpoint. Public RPCs may have rate limits and higher latency.

- **Wallet Security**: Never share your `WALLET_PRIVATE_KEY_B58` or Discord/Telegram tokens. Keep your `config.toml` file secure and never commit it to version control.

- **Start with Small Amounts**: When first using the bot, start with small `BUY_AMOUNT_SOL` values to test your configuration.

- **Simulation Mode**: Use `SIMULATE = true` in your config to test without risking real SOL.

- **Social Media Rate Limits**: Be aware that Discord and Telegram may rate limit if you monitor too many channels or process too many messages.

## Troubleshooting

- **"Durable nonce file not found"**: Run `./setup_nonce.sh` manually
- **"License validation failed"**: Check your `LICENSE_KEY` in config.toml
- **Discord authentication fails**: Verify your Discord token is correct and hasn't expired
- **Telegram login fails**: Check your API credentials and phone number format
- **No signals detected**: Verify scraper sources are configured correctly and messages contain valid token addresses
- **Transactions failing**: Check your wallet has sufficient SOL for trades and fees

## Support

For issues, questions, or feature requests, please contact support through your license provider.

