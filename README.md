# hAI.Files.MD 🇩🇪📚🧠

> Selbstgehosteter Docker-Stack für [files.md](https://github.com/zakirullin/files.md) – optimiert für Portainer und Home-Lab-Umgebungen.

![Status](https://img.shields.io/badge/status-alpha-orange)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Made with Love](https://img.shields.io/badge/made%20with-%E2%9D%A4-red)

---

## 🚀 Idee

Dieses Repository bietet eine minimalistische, aber praxistaugliche Struktur, um `files.md` als
selbstgehosteten Docker-Stack (z. B. über Portainer) zu betreiben.

Frontend (PWA) und Sync-API laufen in **einem einzigen Container** – gebaut direkt aus dem offiziellen Upstream-Repo.

---

## 🧩 Architektur

```text
+-------------------------+
|  Browser (Desktop/Mobile)|
+------------+------------+
             |
             v
   http://SERVER:8080  (PWA + Sync-API)
```

- Ein Container liefert sowohl das Frontend als auch die Sync-API aus.
- Der Build erfolgt direkt aus dem geklonten Upstream-Repo (`/opt/filesmd/files.md`).
- Sync wird aktiviert durch Setzen von `API_URL` in der Umgebung.

---

## 📁 Projektstruktur

```text
hAI.Files.MD/
├── docker-compose.yml      # Stack-Definition
├── README.md               # Diese Datei (DE)
├── README_en.md            # Englische Variante
├── index.html              # Landingpage für GitHub Pages (DE/EN)
├── LICENSE                 # MIT Lizenz
└── app/
    ├── storage/            # Volume für Notizen & Logs (Laufzeit)
    └── tokens/             # Volume für Auth-Tokens (Laufzeit)
```

> Das `files.md`-Upstream-Repo wird **neben** diesem Repo geklont: `/opt/filesmd/files.md`

---

## 🔧 Voraussetzungen

- Docker & Docker Compose
- Portainer (optional, aber empfohlen)
- Git

---

## 🛠️ Setup-Schritte (TL;DR)

1. **Beide Repos klonen** (auf deinem Server):
   ```bash
   sudo mkdir -p /opt/filesmd && cd /opt/filesmd
   git clone https://github.com/zakirullin/files.md.git
   git clone https://github.com/jbkunama1/hAI.Files.MD.git
   mkdir -p hAI.Files.MD/app/storage hAI.Files.MD/app/tokens
   ```

2. **TOKENS_SALT generieren** (einmalig):
   ```bash
   head -c 32 /dev/urandom | base64
   ```
   Den Wert als Umgebungsvariable setzen oder direkt in `docker-compose.yml` bei `TOKENS_SALT=` eintragen.

3. **Stack bauen und starten:**
   ```bash
   cd /opt/filesmd/hAI.Files.MD
   docker compose build --no-cache
   docker compose up -d
   ```

4. **Stack per Portainer deployen:**
   - In Portainer → *Stacks* → *Add stack*
   - Inhalt von `docker-compose.yml` einfügen
   - Umgebungsvariable `TOKENS_SALT` setzen
   - Stack deployen

---

## ⚙️ Umgebungsvariablen

| Variable | Pflicht | Beschreibung |
|---|---|---|
| `APP_URL` | empfohlen | URL der Web-App, z. B. `http://192.168.178.5:8080` |
| `API_URL` | **für Sync nötig** | Aktiviert den Sync-Server – kann gleiche URL wie `APP_URL` sein |
| `STORAGE_DIR` | ja | Pfad für Notizen-Dateien im Container |
| `TOKENS_DIR` | ja | Pfad für Auth-Tokens im Container |
| `TOKENS_SALT` | empfohlen | Zufallswert für Token-Signierung (einmalig generieren) |
| `CERT_DIR` | nein | Pfad zu TLS-Zertifikaten (leer = kein HTTPS) |
| `LOG_FILE` | nein | Pfad zur Log-Datei (default `/tmp/server.log`) |
| `STORAGE_QUOTA_KB` | nein | Speicherlimit pro Nutzer in KB (default 1024 = 1 MB) |
| `BOT_API_TOKEN` | nein | Telegram-Bot-Token (optional, ohne Token = reine Web-App) |

> ⚠️ Ohne `API_URL` läuft der Server nur als Web-App ohne Sync-Funktion!

---

## 🌐 Nutzung

- **Web-App:** `http://DEIN-SERVER:8080`
- **Sync:** läuft automatisch auf demselben Port, sobald `API_URL` gesetzt ist
- In der PWA (DevTools-Konsole) den API-Host setzen:
  ```javascript
  localStorage.setItem('ApiHost', 'http://DEIN-SERVER:8080');
  ```

---

## 🧪 Anpassungen

- **APP_URL / API_URL** auf deine Domain oder LAN-IP setzen
- Port 8080 bei Bedarf ändern (z. B. hinter Reverse Proxy auf 80/443)
- `STORAGE_QUOTA_KB` erhöhen für mehr Speicher pro Nutzer (z. B. `102400` = 100 MB)
- Volumes an dein Backup-Konzept anpassen

---

## 🐛 Problembehandlung

### ❌ `path not found` (Portainer)

Portainer findet den Build-Kontext nicht, weil die Repos noch nicht auf dem Server liegen.

**Lösung:** Repos zuerst klonen (siehe Setup-Schritte oben).
Der `context` in `docker-compose.yml` muss auf den geklonten Upstream-Ordner zeigen:
```yaml
    build:
      context: /opt/filesmd/files.md
      dockerfile: /opt/filesmd/files.md/Dockerfile
```

---

### ❌ Sync funktioniert nicht

Prüfen ob `API_URL` gesetzt ist:
```bash
docker inspect files-md | grep API_URL
```
Ohne `API_URL` startet die Sync-API gar nicht.

---

### ❌ Timeout beim Build / kein Internet aus Docker

```bash
curl -I https://github.com
docker run --rm alpine sh -c "nslookup proxy.golang.org"
```

---

## 📜 Lizenz

Dieses Projekt steht unter der MIT-Lizenz. Details siehe [LICENSE](./LICENSE).

---

## 🌍 Weitere Sprachen

- 🇬🇧 [README_en.md](./README_en.md)
