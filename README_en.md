# hAI.Files.MD 🇬🇧📚🧠

> Self-hosted Docker stack for [files.md](https://github.com/zakirullin/files.md), optimized for Portainer and home lab environments.

![Status](https://img.shields.io/badge/status-alpha-orange)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Made with Love](https://img.shields.io/badge/made%20with-%E2%9D%A4-red)

---

## 🚀 Idea

This repository provides a minimal yet practical structure to run `files.md` as a self-hosted Docker stack (e.g. via Portainer).

Frontend (PWA) and sync API run in a **single container**, built directly from the official upstream repo.

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
- Sync is enabled by setting `API_URL` in the environment.

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
    ├── storage/            # notes & logs volume (runtime)
    └── tokens/             # auth tokens volume (runtime)
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

2. **Generate TOKENS_SALT** (once):
   ```bash
   head -c 32 /dev/urandom | base64
   ```
   Set as environment variable or paste directly into `docker-compose.yml` at `TOKENS_SALT=`.

3. **Build and start the stack:**
   ```bash
   cd /opt/filesmd/hAI.Files.MD
   docker compose build --no-cache
   docker compose up -d
   ```

4. **Deploy via Portainer:**
   - Portainer → *Stacks* → *Add stack*
   - Paste `docker-compose.yml`
   - Set environment variable `TOKENS_SALT`
   - Deploy stack

---

## ⚙️ Environment variables

| Variable | Required | Description |
|---|---|---|
| `APP_URL` | recommended | URL of the web app, e.g. `http://192.168.178.5:8080` |
| `API_URL` | **required for sync** | Enables the sync server – can be the same URL as `APP_URL` |
| `STORAGE_DIR` | yes | Path for notes files inside the container |
| `TOKENS_DIR` | yes | Path for auth tokens inside the container |
| `TOKENS_SALT` | recommended | Random value for token signing (generate once) |
| `CERT_DIR` | no | Path to TLS certificates (empty = no HTTPS) |
| `LOG_FILE` | no | Log file path (default `/tmp/server.log`) |
| `STORAGE_QUOTA_KB` | no | Storage limit per user in KB (default 1024 = 1 MB) |
| `BOT_API_TOKEN` | no | Telegram bot token (optional, without token = web-only) |

> ⚠️ Without `API_URL` the sync API will not start!

---

## 🌐 Usage

- **Web app:** `http://YOUR-SERVER:8080`
- **Sync:** runs automatically on the same port once `API_URL` is set
- Set the API host in the PWA (DevTools console):
  ```javascript
  localStorage.setItem('ApiHost', 'http://YOUR-SERVER:8080');
  ```

---

## 🧪 Customization

- Set **APP_URL / API_URL** to your domain or LAN IP
- Change port 8080 if needed (e.g. behind reverse proxy on 80/443)
- Increase `STORAGE_QUOTA_KB` for more storage per user (e.g. `102400` = 100 MB)
- Adjust volumes to your backup strategy

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

### ❌ Sync not working

Check if `API_URL` is set:
```bash
docker inspect files-md | grep API_URL
```
Without `API_URL` the sync API does not start at all.

---

### ❌ Timeout during build / no internet from Docker

```bash
curl -I https://github.com
docker run --rm alpine sh -c "nslookup proxy.golang.org"
```

---

## 📜 License

This project is licensed under the MIT License. See [LICENSE](./LICENSE).

---

## 🌍 Other languages

- 🇩🇪 [README.md](./README.md)
