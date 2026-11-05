# NFT Gift Showcase — Demo Telegram bot

A working Telegram UI for **clearly labeled, opt-in demos** of gift cards and test Stars values. Not an NFT exchange, payment platform, transfer service, or payout tool.

## Features

- Create a sample card for **any typed gift name or Telegram NFT link** (display-only string).
- Specify **any positive 64-bit integer** test Stars (no balance credit).
- Fully working demo-card buttons: **Accept (demo), Decline, How does it work?**, and view list.
- Telegram admin UI: allow/revoke user IDs, ban/unban, delegate/revoke admins (superadmins only), stats and latest users.
- Telegram Business connection status monitoring; **only** replies to an explicit incoming `демо` or `/demo` keyword in a managed chat; never cold-contacts anyone.
- SQLite for local development and PostgreSQL for Railway; no credentials hardcoded.

## Local run

Python 3.12+:

```bash
cp .env.example .env
# Update BOT_TOKEN and ADMIN_IDS with your own BotFather token and numeric Telegram ID.
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
pip install -r requirements.txt
python -m bot.main
```

Open the bot and use `/start`. The `ADMIN_IDS` owners have access immediately. Everyone else sees their ID and must be approved in `/admin` -> `Разрешить ID`.

Commands: `/start`, `/id`, `/new`, `/admin`, `/cancel`.

## Railway deployment

1. Create your bot via `@BotFather`, retain the API token, and enable **Business Mode** in BotFather if you need the optional integration.
2. Push these files to a private GitHub repository (never commit `.env`).
3. In Railway create a project from the repository. Add the built-in **PostgreSQL** database service.
4. Set service environment variables:
   - `BOT_TOKEN`: token from BotFather.
   - `ADMIN_IDS`: comma-separated numeric IDs for root admins, e.g. `123456789,987654321`.
   - `DATABASE_URL`: `${{Postgres.DATABASE_URL}}` (adjust service name to your Railway Postgres service).
5. Deploy. Railway builds from `Dockerfile` and runs `python -m bot.main` as a continuously running polling worker. **Do not deploy multiple replicas of the same bot token in polling mode.** A public HTTP domain is not required.
6. In Telegram, connect the bot to a Business account under business bot settings, granting the needed right to reply. Business replies work only inside Telegram's permitted chat window (normally 24 hours after the recipient's incoming message).

## Intended demo workflow

The allowlisted operator opens their bot chat, creates a card, and previews it **in their own chat**. The card makes the simulated nature of the Stars amount and NFT completely explicit. Business integrations respond only if a chat participant asks for `демо`; the recipient need not open the bot, but no card or transfer link is pushed to them.

## Safety / scope boundaries

The program does **not** transfer NFT gifts, initiate Stars payments, claim actual payouts, generate phishing pages, gather credentials, access other users' gifts, or perform bulk or unsolicited Business sends. Telegram Business chat access and outgoing rights are limited by official API restrictions.

## Architecture

- Optional user-uploaded photo for any demo gift, otherwise a built-in DEMO graphic.
- `bot/main.py`: aiogram handlers, FSM, authorization and Business updates.
- `bot/storage.py`: persistent SQLAlchemy database model and stats.
- `bot/ui.py`: card display and inline keyboard menus.
- `bot/config.py`: environment variable parsing and database URL normalization.
- `tests/`: smoke/unit tests.

The database schema is auto-created at startup. For a production schema evolution, add Alembic migrations.

To run tests after installing dependencies: `pip install pytest && python -m pytest -q`.
