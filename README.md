# ExitAnt Fibo R-R — LBank

Deploy this automated trading bot to **your own Render account** as a background
worker. The bot runs from a prebuilt Docker image; this repository contains **only
the Render Blueprint (`render.yaml`)** — no source code.

## Deploy to Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-lbank)

```
https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-lbank
```

1. Click the button (or open the link) and sign in to Render.
2. Render reads `render.yaml` and proposes **one background worker** with a
   **1 GB persistent disk** mounted at `/data` (settings and positions survive restarts and redeploys).
3. Enter the environment variables listed below. They are marked `sync: false`, so Render asks you for
   them on the deploy screen and stores them **only in your own Render account**.
4. Click **Apply** and wait for the first deploy to finish.
5. Open Telegram and send **`/start`** to your bot to begin setup.

## Before you deploy

- **LBank account registered with the ExitAnt invite code `64JKW`.** At every start and before every new
  cycle the bot checks this with your own API key; without it, it will not trade.
- **Futures API key** with trading permission (LBank has no API passphrase).
- **Hedge (two-way) position mode** enabled in LBank Futures. The bot refuses to start in one-way mode.
- If the API key has **IP restriction** enabled, whitelist your Render service's outbound IP addresses
  (Render → the service → Settings → Outbound IP addresses). Otherwise LBank rejects the bot's calls with
  error `10022`; on every start the bot tells you the server's current IP to whitelist.
- USDT in your LBank **futures** account.

## Environment variables

| Variable | What to enter |
|---|---|
| `API_KEY` | LBank API key |
| `SECRET_KEY` | LBank API secret |
| `TELEGRAM_BOT_TOKEN` | Bot token issued by [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Your own Telegram chat ID — the bot ignores every other sender |

Nothing else to configure: the exchange, trading mode and state-file location are built into the image.

## What the bot does

Fibo R-R waits for a Bollinger Band breakout on the 5- or 15-minute chart, then enters in up to six steps
(levels) at the spacing you configure, takes profit per level at a Fibonacci ratio towards the opposite
band breakout, keeps one exchange stop-loss for the whole position at your USDT loss limit, and resets the
ladder on an opposite breakout, after a time limit or by hand. A margin guard trims the position when used
margin exceeds your threshold. Every parameter is set in Telegram; twelve languages are supported.

## How it works

- **Prebuilt image.** The worker pulls `docker.io/exitant/exitant-fibo-rr-lbank:latest` from Docker Hub. The image ships
  a compiled binary only — no Python source is included or exposed.
- **Background worker.** There is no web page; the bot runs continuously and talks to you through Telegram.
- **Your keys stay yours.** Secrets are typed into your own Render environment and never leave it.
- **Persistent state.** Settings, positions and pending orders live in a small database on the `/data` disk.

## Important

- **Keep the instance count at 1.** Two instances cause a Telegram `getUpdates` conflict.
- Render **background workers have no free plan** — choose Starter or higher.
- **The bot trades live from the first start.** Begin with a small allocation and watch one full cycle
  (breakout → entries → take-profit/stop-loss) on your account before scaling up.
- After the deploy succeeds, send **`/start`** in Telegram to configure the bot.

---

Blueprint spec: <https://render.com/docs/blueprint-spec> · Prebuilt images: <https://render.com/docs/deploying-an-image>
