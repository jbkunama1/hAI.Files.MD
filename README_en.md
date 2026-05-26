# hAI.Files.MD

Self-hosted instance of [files.md](https://github.com/zakirullin/files.md) — a private, quiet space for your Markdown notes.

## Contents
- [Quickstart](#quickstart)
- [Sync Setup](#sync-setup)
- [Telegram Bot Setup](#telegram-bot-setup)
- [Link a New Device](#link-a-new-device)
- [Docker Compose](#docker-compose)
- [Environment Variables](#environment-variables)
- [Directory Structure](#directory-structure)

---

## Quickstart

```bash
git clone https://github.com/jbkunama1/hAI.Files.MD.git
cd hAI.Files.MD

# Clone upstream files.md (required)
git clone https://github.com/zakirullin/files.md /opt/filesmd/files.md

# Generate salt
export TOKENS_SALT="$(head -c 32 /dev/urandom | base64)"
export BOT_API_TOKEN="your_telegram_bot_token"

# Start
docker compose up -d
```

---

## Sync Setup

files.md supports multiple sync methods:

| Method | Description | Server needed |
|---|---|---|
| **Local-first** | Local only, no sync | No |
| **Cloud folder** | iCloud / Dropbox / Google Drive | No |
| **Self-hosted** | Your own server (this instance) | Yes |
| **Hosted** | `api.files.md` (public) | No |

This instance uses **Self-hosted Sync** via the built-in Go server.

---

## Telegram Bot Setup

The sync login works **exclusively via a Telegram Bot**. Without a bot, no device login is possible.

### Step 1 — Create a Bot

1. Open Telegram → open [@BotFather](https://t.me/BotFather)
2. Send `/newbot`
3. Enter a name, e.g. `MyFilesBot`
4. Enter a username, e.g. `my_files_bot`
5. **Copy the API token** (looks like: `123456789:AAFxxxxxxx`)

### Step 2 — Set Bot Commands (optional but recommended)

In BotFather → "Edit Commands" → paste the following list:

```
chat - 🏠 Home
files - 📄 Files
dirs - 🗂 Dirs
checklists - ☑️ Checklists
schedule - 📆 Schedule
postpone - 🦥 Postpone
rename - ✏️ Rename
move - ➡️ Move
app - 🔗 Open in app
settings - ⚙️ Settings
help - 📕 Help
```

### Step 3 — Add Token to docker-compose.yml

```yaml
environment:
  - BOT_API_TOKEN=123456789:AAFxxxxxxx
  - APP_URL=https://your-domain.com
  - API_URL=https://your-domain.com
  - TOKENS_SALT=${TOKENS_SALT}
  - STORAGE_DIR=/app/storage
  - TOKENS_DIR=/app/tokens
```

### Step 4 — Restart the Container

```bash
docker compose down && docker compose up -d

# Verify bot is running
docker logs files-md --tail 30
```

---

## Link a New Device

Once the bot is running, link a new device like this:

1. **Open your Telegram Bot** (the one you created in Step 1)
2. Send `/app`
3. The bot replies with a **link** → open this link in your browser
4. Device is now linked ✅

Repeat for each additional device (phone, second PC, tablet).

> **Important:** `APP_URL` in `docker-compose.yml` must exactly match the URL
> you use to open the PWA — including `https://` and without a port number.

---

## Docker Compose

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker logs files-md --tail 50

# Rebuild after changes
docker compose up -d --build
```

---

## Environment Variables

| Variable | Description | Example |
|---|---|---|
| `BOT_API_TOKEN` | Telegram Bot Token from BotFather | `123456:AAFxxx` |
| `APP_URL` | PWA URL (must match exactly!) | `https://your-domain.com` |
| `API_URL` | Sync API URL (usually same as APP_URL) | `https://your-domain.com` |
| `TOKENS_SALT` | Random key for signing tokens | `$(head -c 32 /dev/urandom \| base64)` |
| `STORAGE_DIR` | Path to storage directory | `/app/storage` |
| `TOKENS_DIR` | Path to tokens directory | `/app/tokens` |
| `CERT_DIR` | TLS certificate directory (empty = no internal HTTPS) | `/opt/certs` |
| `LOG_FILE` | Path to log file | `/app/storage/server.log` |

---

## Directory Structure

```
hAI.Files.MD/
├── docker-compose.yml    # Docker configuration
├── Dockerfile.sync       # Build file for the sync server
├── .gitignore
├── LICENSE               # MIT
├── README.md             # German documentation
├── README_en.md          # This file (English)
└── index.html            # Landing page / GitHub Pages
```

---

## License

MIT — see [LICENSE](LICENSE)
