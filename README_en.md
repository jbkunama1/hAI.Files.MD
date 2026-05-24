# hAI.Files.MD 🇬🇧📚🧠

> Self-hosted sync server & frontend setup for [files.md](https://github.com/zakirullin/files.md), optimized for Docker, Portainer and home lab environments.

![Status](https://img.shields.io/badge/status-alpha-orange)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Made with Love](https://img.shields.io/badge/made%20with-%E2%9D%A4-red)

---

## 🚀 Idea

This repository provides a minimal yet practical structure to run `files.md` with:

- a **PWA frontend** (browser app)
- a **self-hosted sync server**

as a Docker stack (for example via Portainer).

---

## 🧩 Architecture

```text
+-------------------------+
|  Browser (Desktop/Mobile)|
+------------+------------+
             |
             v
   http://SERVER:8080  (PWA frontend)
             |
             v
   http://SERVER:3333  (sync server)
```

- `files-md-app` → builds the official `files.md` frontend from the upstream repository.
- `files-md-sync` → builds and runs the Go-based sync server.

---

## 📁 Project structure

```text
hAI.Files.MD/
├── docker-compose.yml      # stack definition
├── Dockerfile.sync         # build for sync server
├── README.md               # German
├── README_en.md            # English
├── index.html              # GitHub Pages landing (DE/EN)
├── LICENSE                 # MIT license
├── data/                   # PWA data volume (runtime)
└── app/
    └── storage/            # sync server data (runtime)
```

Note: the actual `files.md` upstream repo is cloned **next to** this project on your server (e.g. `/opt/filesmd/files.md`).

---

## 🔧 Requirements

- Docker & Docker Compose
- Portainer (optional)
- Git

---

## 🛠️ Setup steps (TL;DR)

1. **Clone upstream repo** on your server:
   ```bash
   cd /opt
   mkdir -p filesmd
   cd filesmd
   git clone https://github.com/zakirullin/files.md.git
   ```

2. **Clone this repo** (hAI.Files.MD):
   ```bash
   cd /opt/filesmd
   git clone https://github.com/jbkunama1/hAI.Files.MD.git
   cd hAI.Files.MD
   ```

3. **Generate SALT**:
   ```bash
   head -c 32 /dev/urandom | base64
   ```
   Put the value into `docker-compose.yml` at `SALT=`.

4. **Test stack with Docker Compose:**
   ```bash
   cd /opt/filesmd/hAI.Files.MD
   docker compose build
   docker compose up -d
   ```

5. **Deploy as Portainer stack:**
   - Portainer → *Stacks* → *Add stack*
   - Paste `docker-compose.yml`
   - Deploy stack

---

## 🌐 Usage

### Frontend (PWA)

Open in browser:

```text
http://YOUR-SERVER:8080
```

### Sync server

Sync server runs on:

```text
http://YOUR-SERVER:3333
```

In the `files.md` PWA (DevTools console):

```javascript
localStorage.setItem('ApiHost', 'http://YOUR-SERVER:3333');
```

---

## 🧪 Customization

- Adjust **HOST** in `docker-compose.yml`
- Change ports if needed
- Adjust volumes to your backup/storage strategy

---

## 🐛 Troubleshooting

### ❌ `path "/opt/filesmd/hAI.Files.MD" not found` (Portainer)

Portainer cannot find the build context because the repos have not been cloned to the server yet.

**Fix:** Clone repos on the server first:
```bash
sudo mkdir -p /opt/filesmd
cd /opt/filesmd
git clone https://github.com/zakirullin/files.md.git
git clone https://github.com/jbkunama1/hAI.Files.MD.git
mkdir -p hAI.Files.MD/data hAI.Files.MD/app/storage
```

Then use absolute build context in `docker-compose.yml`:
```yaml
  files-md-sync:
    build:
      context: /opt/filesmd
      dockerfile: /opt/filesmd/hAI.Files.MD/Dockerfile.sync
```

---

### ❌ Timeout during `docker compose build`

The `Dockerfile.sync` tries to clone `github.com/zakirullin/files.md.git` during the build. If the server has no internet access, this causes a timeout.

**Fix:** Use a local build context — copy the upstream folder from the host instead of cloning.

**Step 1 – Clone upstream repo locally** (if not done yet):
```bash
cd /opt/filesmd
git clone https://github.com/zakirullin/files.md.git
```

**Step 2 – Update `Dockerfile.sync`:**
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

**Step 3 – Set build context in `docker-compose.yml` to parent folder:**
```yaml
  files-md-sync:
    build:
      context: /opt/filesmd
      dockerfile: /opt/filesmd/hAI.Files.MD/Dockerfile.sync
```

> ⚠️ `context: /opt/filesmd` is required so that `COPY files.md/sync` works correctly inside the Dockerfile.

---

### ❌ No internet access from Docker build (test)

Check if the server has internet access:
```bash
curl -I https://github.com
docker run --rm alpine sh -c "apk add git && git clone https://github.com/zakirullin/files.md.git /tmp/test"
```

---

## 📜 License

This project is licensed under the MIT License. See [LICENSE](./LICENSE).

---

## 🌍 Other languages

- 🇩🇪 [README.md](./README.md)
