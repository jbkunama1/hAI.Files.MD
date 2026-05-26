# hAI.Files.MD

Eigene Self-hosted Instanz von [files.md](https://github.com/zakirullin/files.md) — ein privater, ruhiger Raum für deine Markdown-Notizen.

## Inhalt
- [Schnellstart](#schnellstart)
- [Sync einrichten](#sync-einrichten)
- [Telegram-Bot einrichten](#telegram-bot-einrichten)
- [Gerät verknüpfen](#gerät-verknüpfen)
- [Docker Compose](#docker-compose)
- [Umgebungsvariablen](#umgebungsvariablen)
- [Verzeichnisstruktur](#verzeichnisstruktur)

---

## Schnellstart

```bash
git clone https://github.com/jbkunama1/hAI.Files.MD.git
cd hAI.Files.MD

# Upstream files.md klonen (Voraussetzung)
git clone https://github.com/zakirullin/files.md /opt/filesmd/files.md

# Salt generieren
export TOKENS_SALT="$(head -c 32 /dev/urandom | base64)"
export BOT_API_TOKEN="dein_telegram_bot_token"

# Starten
docker compose up -d
```

---

## Sync einrichten

files.md unterstützt mehrere Sync-Methoden:

| Methode | Beschreibung | Server nötig |
|---|---|---|
| **Local-first** | Nur lokal, kein Sync | Nein |
| **Cloud-Ordner** | iCloud / Dropbox / Google Drive | Nein |
| **Self-hosted** | Eigener Server (diese Instanz) | Ja |
| **Hosted** | `api.files.md` (öffentlich) | Nein |

Diese Instanz nutzt **Self-hosted Sync** über den integrierten Go-Server.

---

## Telegram-Bot einrichten

Der Sync-Login läuft **ausschließlich über einen Telegram-Bot**. Ohne Bot kein Geräte-Login.

### Schritt 1 — Bot erstellen

1. Telegram öffnen → [@BotFather](https://t.me/BotFather) öffnen
2. `/newbot` eingeben
3. Name vergeben, z. B. `MeineFilesBot`
4. Username vergeben, z. B. `meine_files_bot`
5. **API-Token kopieren** (sieht so aus: `123456789:AAFxxxxxxx`)

### Schritt 2 — Bot-Commands setzen (optional aber empfohlen)

In BotFather → "Edit Commands" → folgende Liste einfügen:

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

### Schritt 3 — Token in docker-compose.yml eintragen

```yaml
environment:
  - BOT_API_TOKEN=123456789:AAFxxxxxxx
  - APP_URL=https://deine-domain.de
  - API_URL=https://deine-domain.de
  - TOKENS_SALT=${TOKENS_SALT}
  - STORAGE_DIR=/app/storage
  - TOKENS_DIR=/app/tokens
```

### Schritt 4 — Container neu starten

```bash
docker compose down && docker compose up -d

# Prüfen ob Bot läuft
docker logs files-md --tail 30
```

---

## Gerät verknüpfen

Sobald der Bot läuft, so wird ein neues Gerät verknüpft:

1. **Telegram-Bot öffnen** (den du in Schritt 1 erstellt hast)
2. `/app` eingeben
3. Bot antwortet mit einem **Link** → diesen Link im Browser öffnen
4. Gerät ist jetzt verknüpft ✅

Jedes weitere Gerät (Handy, zweiter PC, Tablet) genauso verknüpfen.

> **Wichtig:** `APP_URL` in der `docker-compose.yml` muss exakt mit der URL übereinstimmen,
> von der du die PWA aufrufst — inklusive `https://` und ohne Port.

---

## Docker Compose

```bash
# Starten
docker compose up -d

# Stoppen
docker compose down

# Logs anzeigen
docker logs files-md --tail 50

# Neu bauen nach Änderungen
docker compose up -d --build
```

---

## Umgebungsvariablen

| Variable | Beschreibung | Beispiel |
|---|---|---|
| `BOT_API_TOKEN` | Telegram Bot Token von BotFather | `123456:AAFxxx` |
| `APP_URL` | URL der PWA (muss exakt stimmen!) | `https://deine-domain.de` |
| `API_URL` | URL der Sync-API (meist gleich wie APP_URL) | `https://deine-domain.de` |
| `TOKENS_SALT` | Zufälliger Schlüssel zum Signieren von Tokens | `$(head -c 32 /dev/urandom \| base64)` |
| `STORAGE_DIR` | Pfad zum Speicherverzeichnis | `/app/storage` |
| `TOKENS_DIR` | Pfad zum Token-Verzeichnis | `/app/tokens` |
| `CERT_DIR` | TLS-Zertifikat-Verzeichnis (leer = kein HTTPS intern) | `/opt/certs` |
| `LOG_FILE` | Pfad zur Logdatei | `/app/storage/server.log` |

---

## Verzeichnisstruktur

```
hAI.Files.MD/
├── docker-compose.yml    # Docker-Konfiguration
├── Dockerfile.sync       # Build-Datei für den Sync-Server
├── .gitignore
├── LICENSE               # MIT
├── README.md             # Diese Datei (Deutsch)
├── README_en.md          # Englische Dokumentation
└── index.html            # Landingpage / GitHub Pages
```

---

## Lizenz

MIT — siehe [LICENSE](LICENSE)
