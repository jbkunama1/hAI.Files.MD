# hAI.Files.MD 🇬🇧📚🧠

> Self-hosted Docker stack for [files.md](https://github.com/zakirullin/files.md), optimized for Portainer and home lab environments.

![Status](https://img.shields.io/badge/status-alpha-orange)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Made with Love](https://img.shields.io/badge/made%20with-%E2%9D%A4-red)

---

## 🚀 Idea

This repository provides a minimal yet practical structure to run `files.md` as a self-hosted Docker stack (e.g. via Portainer).

Frontend and sync API run in a **single container**, built directly from the official upstream repo.

---

## 🧩 Architecture

```text
+-------------------------+
|  Browser (Desktop/Mobile)|
+------------+------------+
             |
             v
   http://SERVER:8080  (PWA + Sync API)
```

- One container serves both the frontend and the sync API.
- The build uses the locally cloned upstream repo (`/opt/filesmd/files.md`).

---

## 📁 Project structure

```text
hAI.Files.MD/
├── docker-compose.yml      # stack definition
├── README.md               # German
├── README_en.md            # English (this file)
├── index.html              # GitHub Pages landing (DE/EN)
├── LICENSE                 # MIT license
└── app/
    ├── storage/            # notes volume (created at runtime)
    └── tokens/             # auth tokens volume (created at runtime)
```

> The `files.md` upstream repo is cloned **next to** this project: `/opt/filesmd/files.md`

---

## 🔧 Requirements

- Docker & Docker Compose
- Portainer (optional)
- Git

---

## 🛠️ Setup steps (TL;DR)

1. **Clone both repos** on your server:
   ```bash
   sudo mkdir -p /opt/filesmd && cd /opt/filesmd
   git clone https://github.com/zakirullin/files.md.git
   git clone https://github.com/jbkunama1/hAI.Files.MD.git
   mkdir -p hAI.Files.MD/app/storage hAI.Files.MD/app/tokens
   ```

2. **Build and start the stack:**
   ```bash
   cd /opt/filesmd/hAI.Files.MD
   docker compose build --no-cache
   docker compose up -d
   ```

3. **Deploy via Portainer:**
   - Portainer → *Stacks* → *Add stack*
   - Paste `docker-compose.yml`
   - Deploy stack

---

## 🌐 Usage

- Open in browser: `http://YOUR-SERVER:8080`
- Sync API runs on the same port (built-in)

---

## 🧪 Customization

- Set **APP_URL** in `docker-compose.yml` to your domain/IP
- Change port 8080 if needed (e.g. behind reverse proxy on 80/443)
- Adjust volumes to your backup/storage strategy

---

## 🐛 Troubleshooting

### ❌ `path not found` (Portainer)

Portainer cannot find the build context because the repos have not been cloned yet.

**Fix:** Clone repos first (see Setup steps above).
The `context` in `docker-compose.yml` must point to the cloned upstream folder:
```yaml
    build:
      context: /opt/filesmd/files.md
      dockerfile: /opt/filesmd/files.md/Dockerfile
```

---

### ❌ Timeout during build

The build only reads local files from `/opt/filesmd/files.md` — no internet required.
If `go mod download` times out, check Docker DNS:
```bash
docker run --rm alpine sh -c "nslookup proxy.golang.org"
```

---

### ❌ No internet access from Docker build (test)

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
