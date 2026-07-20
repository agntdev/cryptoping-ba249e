# CryptoPing — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for personalized cryptocurrency price alerts and notifications. Users track price thresholds, percentage changes, and receive optional morning summaries with quiet hours and deduplication. Owner gets aggregated usage stats without exposing user data.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- retail Telegram users tracking crypto prices
- bot owner needing analytics

## Success criteria

- Users can set up and manage price alert rules
- Notifications are delivered according to configured thresholds and quiet hours
- Owner receives aggregated usage statistics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open onboarding and main menu
- **/price** (command, actor: user, command: /price) — Check current price of a specific coin or full list
  - inputs: ticker symbol or 'all'
  - outputs: current price and change
- **Add coin** (button, actor: user, callback: crypto:add) — Add cryptocurrency to monitoring list
  - inputs: ticker symbol
  - outputs: confirmation of added coin
- **Create alert** (button, actor: user, callback: alert:create) — Configure price threshold or percentage change alert

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message
2. Interactive crypto selection buttons
3. Time zone setup
4. Quiet hours configuration

_Data touched:_ User

### Price Alert Setup
_Trigger:_ crypto:add

1. Select coin
2. Choose alert type (price threshold or percentage change)
3. Enter parameters
4. Confirm rule

_Data touched:_ AlertRule, User

### Morning Summary
_Trigger:_ scheduled time

1. Check user's time setting
2. Fetch prices for all tracked coins
3. Format summary with changes
4. Send to user

_Data touched:_ User, Ticker

### Error Handling
_Trigger:_ API failure

1. Log error
2. Retry with backoff
3. Notify user on next interaction
4. Pause alerts temporarily

_Data touched:_ PriceEvent, User

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user preferences and settings
  - fields: Telegram ID, Time zone, Quiet hours (start/end), Monitoring list, Alert rules, Morning summary time
- **Ticker** _(retention: session)_ — Cryptocurrency price data
  - fields: Symbol, Name, Last known price, Source metadata
- **AlertRule** _(retention: persistent)_ — User-defined alert configuration
  - fields: Type (price threshold/percentage), Target value/percentage, Active status, Last triggered time
- **PriceEvent** _(retention: session)_ — Price change tracking
  - fields: Old value, New value, Change percentage, Timestamp, Triggered rules
- **OwnerStats** _(retention: persistent)_ — Aggregated usage statistics
  - fields: User count, Active rules count, Trigger counts by ticker

## Integrations

- **Telegram** (required) — Messaging and user interaction
- **Price API** (required) — Cryptocurrency price data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /admin_stats - View aggregated usage statistics and top tickers

## Notifications

- Price threshold alerts
- Percentage change alerts
- Morning price summaries
- Error notifications
- Admin statistics

## Permissions & privacy

- All user data is private and stored securely
- No personal data shared with third parties
- Aggregated stats exclude identifiable information

## Edge cases

- Typo in ticker symbol (suggest corrections)
- API price source failure (silent retries)
- Quiet hours overlapping alert triggers (defer/discard)
- Multiple alerts triggered simultaneously (deduplication)

## Required tests

- Verify alert deduplication during price fluctuations
- Test morning summary formatting with multiple currencies
- Validate quiet hours behavior with time zone changes
- Confirm admin stats aggregation without personal data leakage

## Assumptions

- Default price check frequency is 1 minute
- Default alert pause after trigger is 6 hours
- Morning summary defaults to 1h/24h change periods
- Admin stats exclude personal user data
