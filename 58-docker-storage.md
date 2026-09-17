# Lab 58: Docker Storage Architecture & Management

## 📌 Objective

Understand the Docker storage architecture, differentiate between ephemeral container layers (Copy-on-Write) and persistent storage strategies (Volumes, Bind Mounts, and tmpfs), and master Docker volume management commands.

---

## 🧠 Docker Storage Architecture

By default, all files created inside a container are stored on a **writable container layer**. When the container is deleted (`docker rm`), this layer is destroyed and all data is permanently lost.

To persist data beyond container lifecycle, Docker provides three persistent storage mechanisms:

```text
┌─────────────────────────────────────────────────────────────┐
│                         HOST SYSTEM                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. DOCKER VOLUMES                                         │
│      Stored in Docker's managed storage area:               │
│      /var/lib/docker/volumes/<volume_name>/_data            │
│      (Managed exclusively by Docker CLI & Engine)           │
│                                                             │
│   2. BIND MOUNTS                                            │
│      Can be anywhere on the host filesystem:                │
│      /home/user/project or C:\Users\danis\Desktop\...       │
│      (Controlled and modified directly by host processes)   │
│                                                             │
│   3. TMPFS MOUNTS                                           │
│      Stored purely in the host system's RAM memory          │
│      (Never written to disk; lost when container stops)     │
│                                                             │
└──────────────────────────────┬──────────────────────────────┘
                               │ Mount point (-v / --mount)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                      CONTAINER LAYER                        │
│          /var/lib/postgresql/data  OR  /usr/local/apache/   │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Comparison: Storage Types

| Feature | Named Volumes | Bind Mounts | tmpfs Mounts |
|---------|---------------|-------------|--------------|
| **Host Location** | Managed by Docker (`/var/lib/docker/volumes/`) | Arbitrary host directory chosen by user | Host memory (RAM) |
| **Portability** | High (works identically on Linux, Windows, macOS) | Medium (depends on host directory structure) | Medium (Linux only) |
| **Ease of Backup** | Easy via Docker CLI commands | Standard host backup utilities | N/A (non-persistent) |
| **Best Used For** | Production databases, shared data across containers | Local development, source code sync, configs | Sensitive data, temporary high-speed scratchpads |

---

## 🛠️ Step-by-Step Volume Management Commands

### 1. Create a Named Volume

```bash
docker volume create my-app-data
```

---

### 2. List All Volumes

```bash
docker volume ls
```

**Output:**
```text
DRIVER    VOLUME NAME
local     my-app-data
```

---

### 3. Inspect Volume Metadata

Examine the exact storage path and driver parameters:

```bash
docker volume inspect my-app-data
```

**Sample Output:**
```json
[
    {
        "CreatedAt": "2026-09-17T10:45:00Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-app-data/_data",
        "Name": "my-app-data",
        "Options": null,
        "Scope": "local"
    }
]
```

---

### 4. Mount Volume to a Container

Run an Ubuntu container with the volume mounted to `/data`:

```bash
docker run -it --rm -v my-app-data:/data ubuntu bash
```

Inside the container, create a persistent file:
```bash
echo "Docker persistent storage verified!" > /data/test.txt
exit
```

Now launch a completely new container with the same volume to verify data persistence:
```bash
docker run -it --rm -v my-app-data:/data ubuntu cat /data/test.txt
```

**Output:**
```text
Docker persistent storage verified!
```
Even though the first container was deleted (`--rm`), the data persisted in the volume!

---

### 5. Remove Volumes

Remove a specific volume (must not be in use by any container):
```bash
docker volume rm my-app-data
```

Delete all unused volumes at once:
```bash
docker volume prune -f
```

---

## 🧾 Storage Commands Reference

| Command | Description |
|---------|-------------|
| `docker volume create <name>` | Create a named volume |
| `docker volume ls` | List all local volumes |
| `docker volume inspect <name>` | View detailed JSON configuration and host mountpoint |
| `docker volume rm <name>` | Delete a specific volume |
| `docker volume prune` | Clean up all unattached/dangling volumes |
| `-v <volume>:<container_path>` | Shorthand syntax to mount a volume |
| `--mount type=volume,source=...,target=...` | Explicit syntax recommended for production |

---

## 📝 Key Takeaways

1. Volumes are the **preferred mechanism** for persisting data generated by and used by Docker containers.
2. Containers are ephemeral by default — stateful services (databases, file uploads) must always use volumes or bind mounts.
3. Volumes isolate the container's storage lifecycle from the host filesystem permissions and OS idiosyncrasies.
