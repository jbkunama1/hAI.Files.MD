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

Frontend und Sync-API laufen in **einem einzigen Container** – gebaut direkt aus dem offiziellen Upstream-Repo.

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
    ├── storage/            # Volume für Notizen (wird zur Laufzeit angelegt)
    └── tokens/             # Volume für Auth-Tokens (wird zur Laufzeit angelegt)
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

2. **Stack bauen und starten:**
   ```bash
   cd /opt/filesmd/hAI.Files.MD
   docker compose build --no-cache
   docker compose up -d
   ```

3. **Stack per Portainer deployen:**
   - In Portainer → *Stacks* → *Add stack*
   - Inhalt von `docker-compose.yml` einfügen
   - Stack deployen

---

## 🌐 Nutzung

- Aufruf im Browser: `http://DEIN-SERVER:8080`
- Sync-API läuft auf demselben Port (eingebaut)

---

## 🧪 Anpassungen

- **APP_URL** in `docker-compose.yml` auf deine Domain/IP setzen
- Port 8080 bei Bedarf ändern (z. B. hinter Reverse Proxy auf 80/443)
- Volumes an dein Backup-/Storage-Konzept anpassen

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

### ❌ Timeout beim Build

Der Build liest nur lokale Dateien aus `/opt/filesmd/files.md` – kein Internet nötig.
Wenn `go mod download` timeoutet, Docker-DNS prüfen:
```bash
docker run --rm alpine sh -c "nslookup proxy.golang.org"
```

---

### ❌ Kein Internet aus Docker-Build heraus (Test)

```bash
curl -I https://github.com
docker run --rm alpine sh -c "apk add git && git clone https://github.com/zakirullin/files.md.git /tmp/test"
```

---

## 📜 Lizenz

Dieses Projekt steht unter der MIT-Lizenz. Details siehe [LICENSE](./LICENSE).

---

## 🌍 Weitere Sprachen

- 🇬🇧 [README_en.md](./README_en.md)
