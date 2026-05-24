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

- Adjust **HOST** in `docker-compose.yml` and `Dockerfile.sync`
- Change ports if needed
- Adjust volumes to your backup/storage strategy

---

## 📜 License

This project is licensed under the MIT License. See [LICENSE](./LICENSE).

---

## 🌍 Other languages

- 🇩🇪 [README.md](./README.md)
