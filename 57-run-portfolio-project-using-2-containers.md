# Lab 57: Run Portfolio Project Using 2 Containers

## 📌 Objective

Deploy a multi-tier containerized architecture for a web portfolio application using two interconnected Docker containers:
1. **Frontend Web Server Container** (Apache / Nginx) serving static portfolio web assets.
2. **Backend Database Container** (PostgreSQL / MySQL) for dynamic contact form submissions or guestbook logging.
3. Connected over an isolated user-defined Docker bridge network with internal DNS communication.

---

## 🏗️ Architecture Diagram

```text
       Browser (Host Machine)
                 │
                 │ http://localhost:8080
                 ▼
     ┌───────────────────────┐
     │ Host Port: 8080       │
     └───────────┬───────────┘
                 │ (-p 8080:80)
                 ▼
┌─────────────────────────────────────────────────────────────┐
│               Custom Docker Network (portfolio-net)         │
│                                                             │
│   ┌──────────────────────────┐    Internal DNS    ┌──────────────────────┐
│   │   Frontend Container     │   "portfolio-db"   │   Backend Database   │
│   │   (portfolio-web)        ├───────────────────►│   (portfolio-db)     │
│   │   Apache / Nginx         │      Port 5432     │   PostgreSQL / Data  │
│   │   Port 80                │                    │   Volume: pg-data    │
│   └────────────┬─────────────┘                    └──────────────────────┘
│                │                                              │
│                ▼                                              ▼
│         Mounted Assets                                Persistent Storage
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Implementation

### Step 1: Create an Isolated User-Defined Bridge Network

```bash
docker network create portfolio-net
```

Verify network status:
```bash
docker network ls
```

---

### Step 2: Launch Container 1 — Backend Database (PostgreSQL)

Deploy the database container attached to `portfolio-net`:

```bash
docker run -d \
  --name portfolio-db \
  --network portfolio-net \
  -e POSTGRES_DB=portfolio_db \
  -e POSTGRES_USER=danish \
  -e POSTGRES_PASSWORD=securepassword123 \
  -v portfolio-db-data:/var/lib/postgresql/data \
  postgres:16-alpine
```

**Flag Explanations:**
- `--network portfolio-net`: Attaches the container to the custom bridge network.
- `-e POSTGRES_*`: Injects database credentials securely.
- `-v portfolio-db-data:/var/lib/postgresql/data`: Ensures database records survive container restarts.
- Note: We do **not** need `-p` on the database because only the frontend container needs to reach it inside `portfolio-net`!

---

### Step 3: Launch Container 2 — Frontend Web Server (Portfolio Web)

Deploy the web container on the same network with port forwarding enabled for external browser access:

```bash
docker run -d \
  --name portfolio-web \
  --network portfolio-net \
  -p 8080:80 \
  -v /mnt/c/Users/danis/Desktop/Devops/my_portfolio_final/dist:/usr/local/apache2/htdocs/ \
  httpd:alpine
```

**Flag Explanations:**
- `-p 8080:80`: Forwards host port 8080 to container port 80.
- `-v .../dist:/usr/local/apache2/htdocs/`: Mounts your local production portfolio files directly into Apache's document root.
- `--network portfolio-net`: Allows `portfolio-web` to resolve and communicate with `portfolio-db`.

---

### Step 4: Verify Both Containers are Running

```bash
docker container ls
```

**Sample Output:**
```text
CONTAINER ID   IMAGE                COMMAND                  STATUS         PORTS                  NAMES
b3f12a4567cd   httpd:alpine         "httpd-foreground"       Up 45 seconds  0.0.0.0:8080->80/tcp   portfolio-web
c894ef0123ab   postgres:16-alpine   "docker-entrypoint.s…"   Up 1 minute    5432/tcp               portfolio-db
```

---

### Step 5: Test Inter-Container Communication & DNS

Execute a command inside the frontend container to verify network connectivity to `portfolio-db`:

```bash
docker exec -it portfolio-web ping -c 3 portfolio-db
```

Or verify database port reachability:
```bash
docker exec -it portfolio-web nc -zv portfolio-db 5432
```

**Expected Output:**
```text
portfolio-db (172.19.0.2:5432) open
```

---

### Step 6: Verify in Browser

1. Open your browser on Windows.
2. Navigate to: `http://localhost:8080`
3. Verify that your full portfolio application is rendered seamlessly.

---

## 🧹 Cleanup Instructions

```bash
# Stop and remove containers
docker rm -f portfolio-web portfolio-db

# Remove the custom network
docker network rm portfolio-net

# (Optional) Remove database volume if no longer needed
docker volume rm portfolio-db-data
```

---

## 📝 Key Takeaways

1. **Defense in Depth / Security**: Only expose frontend ports (`-p 8080:80`) to the host network. Keep the database container private inside the bridge network.
2. **Built-in Service Discovery**: Containers on custom networks resolve one another by name (e.g. `portfolio-db`) rather than fluctuating IP addresses.
3. **Multi-tier Decoupling**: Frontend presentation logic is fully separated from backend persistent storage, matching production microservice architecture patterns.
