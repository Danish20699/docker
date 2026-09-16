# Lab 54: Docker httpd Container Run with Port Forwarding

## 📌 Objective

Run an Apache (httpd) container with port forwarding (`-p`) to access the web server from your browser via `localhost`.

---

## 🧠 What is Port Forwarding?

Containers have their own **private network** — your browser can't reach them directly. Port forwarding creates a **door** between your machine and the container.

```
Your Browser                           Container
     │                                      │
     │  visits localhost:8080               Apache listening on port 80
     │                                      │
     └────────── -p 8080:80 ───────────────┘
                 (port forwarding)
```

### Format

```
-p <host-port>:<container-port>
    ──────────   ──────────────
        │              │
        │              └── Port INSIDE the container (Apache uses 80)
        │
        └── Port on YOUR machine (what you type in browser)
```

---

## 🔧 Step 1 — Pull the Apache Image

```bash
docker pull httpd
```

---

## ▶️ Step 2 — Run httpd with Port Forwarding

```bash
docker run -d --name httpd-port-fwd -p 8080:80 httpd
```

### Command Breakdown

| Part | What It Does |
|------|-------------|
| `-d` | Run in background (detached) |
| `--name httpd-port-fwd` | Name the container |
| `-p 8080:80` | Forward host port 8080 → container port 80 |
| `httpd` | Apache image |

---

## ✅ Step 3 — Verify Container & Port Mapping

```bash
docker container ls
```

```
CONTAINER ID   IMAGE   COMMAND              STATUS       PORTS                  NAMES
xxxxxxxxxxxx   httpd   "httpd-foreground"   Up 5 sec     0.0.0.0:8080->80/tcp   httpd-port-fwd
```

The **PORTS** column shows: `0.0.0.0:8080->80/tcp`

This means:
- `0.0.0.0` = listening on all network interfaces
- `8080->80/tcp` = host port 8080 is forwarded to container port 80

---

## 🌐 Step 4 — Access in Browser

Open: **http://localhost:8080**

Result: **"It works!"** ✅

---

## 🔄 Step 5 — Try a Different Port

Stop the container and run on port **9090** instead:

```bash
docker rm -f httpd-port-fwd
docker run -d --name httpd-port-fwd -p 9090:80 httpd
```

Open: **http://localhost:9090** → Same **"It works!"** but on a different port! ✅

This proves **you can choose any available port** on your host machine.

---

## 📊 Port Forwarding Examples

| Command | Browser URL | What Happens |
|---------|------------|-------------|
| `-p 8080:80` | `localhost:8080` | Host 8080 → Container 80 |
| `-p 9090:80` | `localhost:9090` | Host 9090 → Container 80 |
| `-p 80:80` | `localhost` | Host 80 → Container 80 (default HTTP port) |
| `-p 3000:80` | `localhost:3000` | Host 3000 → Container 80 |

---

## ⚠️ Common Errors

### Port Already Allocated

```
Error: Bind for 0.0.0.0:8080 failed: port is already allocated
```

**Cause:** Another container is already using that port.

**Fix:** Remove the old container first:

```bash
docker rm -f <old-container-name>
```

Or use a different port:

```bash
docker run -d -p 9090:80 httpd
```

---

## 🧹 Cleanup

```bash
docker rm -f httpd-port-fwd
```

---

## 🧾 Commands Used in This Lab

| # | Command | Description |
|---|---------|-------------|
| 1 | `docker pull httpd` | Download the Apache image |
| 2 | `docker run -d --name httpd-port-fwd -p 8080:80 httpd` | Run Apache with port forwarding |
| 3 | `docker container ls` | Verify container is running and check PORTS column |
| 4 | `docker rm -f httpd-port-fwd` | Force stop and remove the container |
| 5 | `docker run -d --name httpd-port-fwd -p 9090:80 httpd` | Run on a different port |

---

## 📝 Key Takeaways

1. **`-p` (port forwarding)** maps a host port to a container port — without it, the browser can't reach the container.
2. **Left side** = your machine's port (you choose). **Right side** = container's port (fixed by the service).
3. **`0.0.0.0:8080->80/tcp`** in the PORTS column confirms port forwarding is active.
4. **Only one container** can use a host port at a time. If it's taken, use a different port or remove the old container.
5. The container's internal port (80) never changes — you only change the host port.
