# Robux Earnings Bot — Bot specification

**Archetype:** custom

**Voice:** playful and encouraging — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot where users earn 0.01 Robux per click, track balances, and request manual withdrawals. Admins process payouts via Roblox usernames with audit trails and rate-limited click protection.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Roblox players
- Community members

## Success criteria

- Users earn Robux via clicks
- Admins process withdrawals efficiently
- Fraudulent activity detected and blocked

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu
- **Click to Earn** (button, actor: user, callback: click:start) — Add 0.01 Robux to balance with rate-limited debounce
- **Balance** (button, actor: user, callback: balance:view) — Show current Robux balance and click count
- **Withdraw** (button, actor: user, callback: withdraw:start) — Initiate withdrawal request
- **Link Roblox** (button, actor: user, callback: link:start) — Set/update Roblox username for withdrawals
- **/admin** (command, actor: admin, command: /admin) — Open admin dashboard for withdrawal processing

## Flows

### Click Earnings
_Trigger:_ button:click:start

1. Validate rate limit (1 click/sec)
2. Add 0.01 Robux to balance
3. Log click event
4. Show updated balance

_Data touched:_ User, ClickEvent

### Withdrawal Request
_Trigger:_ button:withdraw:start

1. Validate minimum balance
2. Collect Roblox username
3. Create withdrawal request
4. Notify admin
5. Show request status to user

_Data touched:_ WithdrawalRequest

### Admin Processing
_Trigger:_ command:/admin

1. List pending withdrawals
2. Approve/reject requests
3. Update notes
4. Notify user of status change

_Data touched:_ WithdrawalRequest, User

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with Robux balance and linked Roblox username
  - fields: telegram_id, display_name, roblox_username, balance, registration_ts
- **ClickEvent** _(retention: persistent)_ — Record of individual clicks for audit and rate-limiting
  - fields: user_id, timestamp, amount
- **WithdrawalRequest** _(retention: persistent)_ — Pending or processed withdrawal request
  - fields: user_id, amount, roblox_username, status, request_ts, admin_notes

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure minimum withdrawal amount
- Set rate-limit interval
- Define admin notification group

## Notifications

- Admin alerts on new withdrawals
- User notifications on withdrawal status changes
- Rate-limit warnings

## Permissions & privacy

- Roblox username linkage optional until withdrawal
- All balances stored securely
- Admins can view linked usernames

## Edge cases

- Withdrawal below minimum amount
- Invalid Roblox username format
- Rate-limit violations
- Admin action without linked username

## Required tests

- Verify 0.01 Robux added per click with rate limit
- Validate withdrawal workflow from request to admin approval
- Test admin dashboard filtering of pending requests

## Assumptions

- Minimum withdrawal default 10 Robux
- Click rate limit 1/sec
- Admins manually process Roblox transfers
