# hAI.Files.MD 🇩🇪📚🧠

> Selbstgehosteter Sync-Server & Frontend-Setup für [files.md](https://github.com/zakirullin/files.md) – optimiert für Docker, Portainer und Home-Lab-Umgebungen.

![Status](https://img.shields.io/badge/status-alpha-orange)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Made with Love](https://img.shields.io/badge/made%20with-%E2%9D%A4-red)

---

## 🚀 Idee

Dieses Repository bietet eine minimalistische, aber praxistaugliche Struktur, um `files.md` mit

- einem **PWA-Frontend** (Browser-App)
- einem **self-hosted Sync-Server**

als Docker-Stack (z. B. über Portainer) zu betreiben.

Die Konfiguration ist bewusst einfach gehalten und kann direkt in ein Homelab-Setup übernommen werden.

---

## 🧩 Architektur

```text
+-------------------------+
|  Browser (Desktop/Mobile)|
+------------+------------+
             |
             v
   http://SERVER:8080  (PWA Frontend)
             |
             v
   http://SERVER:3333  (Sync-Server)
```

- `files-md-app` → baut das offizielle `files.md`-Frontend aus dem Upstream-Repository.
- `files-md-sync` → baut und startet den Go-basierten Sync-Server.

---

## 📁 Projektstruktur

```text
hAI.Files.MD/
├── docker-compose.yml      # Stack-Definition
├── Dockerfile.sync         # Build für Sync-Server
├── README.md               # Diese Datei (DE)
├── README_en.md            # Englische Variante
├── index.html              # Landingpage für GitHub Pages (DE/EN)
├── LICENSE                 # MIT Lizenz
├── data/                   # Volume für PWA-Daten (wird zur Laufzeit angelegt)
└── app/
    └── storage/            # Volume für Sync-Server-Daten (wird zur Laufzeit angelegt)
```

> Hinweis: Das eigentliche `files.md`-Upstream-Repo wird **neben** diesem Repo geklont (z. B. nach `/opt/filesmd/files.md`).

---

## 🔧 Voraussetzungen

- Docker & Docker Compose
- Portainer (optional, aber empfohlen)
- Git für das Klonen von Repositories

---

## 🛠️ Setup-Schritte (TL;DR)

1. **Upstream-Repo klonen** (auf deinem Server):
   ```bash
   cd /opt
   mkdir -p filesmd
   cd filesmd
   git clone https://github.com/zakirullin/files.md.git
   ```

2. **Dieses Repo klonen** (hAI.Files.MD):
   ```bash
   cd /opt/filesmd
   git clone https://github.com/jbkunama1/hAI.Files.MD.git
   cd hAI.Files.MD
   ```

3. **SALT setzen** (einmalig generieren):
   ```bash
   head -c 32 /dev/urandom | base64
   ```
   Den Wert in `docker-compose.yml` bei `SALT=` eintragen.

4. **Stack mit Docker Compose testen:**
   ```bash
   cd /opt/filesmd/hAI.Files.MD
   docker compose build
   docker compose up -d
   ```

5. **Stack per Portainer deployen:**
   - In Portainer → *Stacks* → *Add stack*
   - Inhalt von `docker-compose.yml` einfügen
   - Stack deployen

---

## 🌐 Nutzung

### Frontend (PWA)

- Aufruf im Browser:

  ```
  http://DEIN-SERVER:8080
  ```

### Sync-Server

- Der Sync-Server läuft auf:

  ```
  http://DEIN-SERVER:3333
  ```

- In der `files.md`-PWA (DevTools-Konsole):

  ```javascript
  localStorage.setItem('ApiHost', 'http://DEIN-SERVER:3333');
  ```

---

## 🧪 Anpassungen

- **HOST** in `docker-compose.yml`
- Ports bei Bedarf ändern (z. B. 8080 → 80 hinter Reverse Proxy)
- Volumes an dein Backup-/Storage-Konzept anpassen

---

## 🐛 Problembehandlung

### ❌ `path "/opt/filesmd/hAI.Files.MD" not found` (Portainer)

Portainer findet den Build-Kontext nicht, weil die Repos noch nicht auf dem Server liegen.

**Lösung:** Repos zuerst auf dem Server klonen:
```bash
sudo mkdir -p /opt/filesmd
cd /opt/filesmd
git clone https://github.com/zakirullin/files.md.git
git clone https://github.com/jbkunama1/hAI.Files.MD.git
mkdir -p hAI.Files.MD/data hAI.Files.MD/app/storage
```

Danach in `docker-compose.yml` absoluten Build-Kontext verwenden:
```yaml
  files-md-sync:
    build:
      context: /opt/filesmd
      dockerfile: /opt/filesmd/hAI.Files.MD/Dockerfile.sync
```

---

### ❌ Timeout beim `docker compose build`

Das `Dockerfile.sync` versucht `github.com/zakirullin/files.md.git` während des Builds zu klonen. Wenn der Server keinen ausreichenden Internetzugang hat, führt das zu einem Timeout.

**Lösung:** Lokalen Build-Kontext verwenden – der Upstream-Ordner wird vom Host ins Image kopiert statt geclont.

**Schritt 1 – Upstream-Repo lokal klonen** (falls noch nicht geschehen):
```bash
cd /opt/filesmd
git clone https://github.com/zakirullin/files.md.git
```

**Schritt 2 – `Dockerfile.sync` anpassen:**
```dockerfile
# syntax=docker/dockerfile:1

FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY files.md/sync ./sync

WORKDIR /app/sync

RUN go mod download && go build -o /app/files-md-sync

FROM alpine:3.19

WORKDIR /app

RUN adduser -D -h /app filesmd

COPY --from=builder /app/files-md-sync /app/files-md-sync

RUN mkdir -p /app/storage && chown -R filesmd:filesmd /app

USER filesmd

ENV HOST=0.0.0.0
ENV PORT=3333
ENV SALT=""

EXPOSE 3333

CMD ["/app/files-md-sync"]
```

**Schritt 3 – `docker-compose.yml` Build-Kontext auf übergeordneten Ordner setzen:**
```yaml
  files-md-sync:
    build:
      context: /opt/filesmd
      dockerfile: /opt/filesmd/hAI.Files.MD/Dockerfile.sync
```

> ⚠️ `context: /opt/filesmd` ist zwingend nötig, damit `COPY files.md/sync` im Dockerfile funktioniert.

---

### ❌ Kein Internet aus Docker-Build heraus (Test)

Prüfen ob der Server Internetzugang hat:
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
