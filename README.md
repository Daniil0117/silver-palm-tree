# STRIKGOLD FC 27 shop — Railway + YooKassa (RUB)

Starter project: Express, SQLite orders, basic admin API, Telegram notifications and YooKassa redirect checkout. It is not deployed by this archive; you must supply merchant credentials and deploy it yourself.

## Requirements
- Node.js 20+
- Railway account (or local Node.js for testing)
- YooKassa merchant account approved for your business and products
- Telegram bot (optional)

## Local start
1. `npm install`
2. Copy `.env.example` to `.env` and set `ADMIN_PASSWORD`, a random `SESSION_SECRET`, and `DATABASE_DIR=./data` for local testing.
3. Add your own YooKassa Shop ID and secret key only after the merchant account is approved. Never commit `.env`.
4. `npm start`
5. Open `http://localhost:3000`. The `/api/health` endpoint should return `{ "ok": true }`.

Without both `YOOKASSA_SHOP_ID` and `YOOKASSA_SECRET_KEY`, checkout remains disabled and the order is stored as an unconfigured request. No credentials are bundled.

## Railway deploy
1. Put the project in a private GitHub repository. Ensure `.env` is not committed.
2. Railway → New Project → Deploy from GitHub Repo; select the repository.
3. Service Settings: start command `npm start` (Railway usually detects the Node start script automatically).
4. Add a Volume attached to this service with mount path `/app/data`. Keep `DATABASE_DIR=/app/data` in Variables. The database file will be `/app/data/strikgold.sqlite`.
5. Generate a Railway public domain in Networking. Set `BASE_URL` to `https://YOUR-RAILWAY-DOMAIN` with no trailing slash, then redeploy.
6. Add variables: `CURRENCY=RUB`, `DATABASE_DIR=/app/data`, `ADMIN_PASSWORD`, `SESSION_SECRET`, `YOOKASSA_SHOP_ID`, `YOOKASSA_SECRET_KEY`, and optionally `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`.
7. Replace demo `PRICE_*` values. These are in kopecks: `49900` means 499.00 RUB.
8. Deploy and check `https://YOUR-RAILWAY-DOMAIN/api/health`.
9. In YooKassa merchant settings, configure HTTP notifications/webhook URL `https://YOUR-RAILWAY-DOMAIN/api/yookassa-webhook` for `payment.succeeded`. The URL must be HTTPS. This endpoint re-fetches the payment from YooKassa API with merchant credentials and verifies status, paid flag, order ID, payment ID, amount and currency before marking an order paid.
10. Test with the provider's available test mode/test credentials first, then perform a controlled real payment only after the merchant and business model are approved.

## Telegram
1. Create a bot with `@BotFather`; store its token in Railway Variables as `TELEGRAM_BOT_TOKEN`.
2. Send the bot a message from the target account/chat. Obtain the correct `chat_id` using Telegram Bot API `getUpdates` or an established bot utility.
3. Set `TELEGRAM_CHAT_ID` in Railway Variables and redeploy.

## Important implementation notes
- YooKassa API calls are server-side only. Credentials must never be exposed to browser code.
- A success return URL is not proof of payment. Only a verified `payment.succeeded` webhook changes the order to `paid`.
- The webhook handler is idempotent for repeated notifications.
- The database is persisted only if the Railway Volume is mounted at `/app/data` and `DATABASE_DIR` matches it.
- The admin API is a starter, not a complete hardened admin portal: sessions are in-memory and reset on redeploy; add rate limiting, stronger session handling, CSRF protections where applicable, monitoring, backups and security review before public production use.
- Prices are demo values. Confirm merchant eligibility, legal/tax requirements and the payment provider's approval before accepting live payments.
