# 🐳 Docker Commands Reference — Labs 48–51

A quick reference of all Docker commands covered during the hands-on labs, along with descriptions and flag breakdowns.

---

## 📦 Image Commands

| # | Command | Description |
|---|---------|-------------|
| 1 | `docker images` | Lists all Docker images stored locally on your machine. Also known as `docker image ls`. |
| 2 | `docker pull <image>` | Downloads an image from Docker Hub to your local machine without running it. |

---

## ▶️ Container Run Commands

| # | Command | Description |
|---|---------|-------------|
| 3 | `docker run hello-world` | Pulls the `hello-world` image (if not found locally), creates a container, runs it, prints a success message, and exits. Used to verify Docker is installed correctly. |
| 4 | `docker run -it ubuntu /bin/bash` | Runs an Ubuntu container in **interactive mode** with a live bash terminal. You get a root shell inside the container. The container stops when you type `exit`. |
| 5 | `docker run -d --name my-ubuntu ubuntu sleep infinity` | Runs a container in **detached (background) mode** with a custom name `my-ubuntu`. The `sleep infinity` command keeps the container alive so you can `exec` into it later. |

---

## 🔧 Container Management Commands

| # | Command | Description |
|---|---------|-------------|
| 6 | `docker exec -it <name> /bin/bash` | Opens a bash shell **inside an already running** container. Unlike `docker run`, this does NOT create a new container. The container must be in a running state. |
| 7 | `docker stop <name or id>` | Gracefully stops a running container. Sends a SIGTERM signal followed by SIGKILL after a timeout. |
| 8 | `docker rm <name or id>` | Permanently deletes a stopped container. Cannot remove a running container (stop it first). |

---

## 🔍 Inspection Commands

| # | Command | Description |
|---|---------|-------------|
| 9 | `docker container ls` | Lists only **currently running** containers. Alias: `docker ps`. |
| 10 | `docker container ls -a` | Lists **all** containers — both running and stopped. The `-a` flag means "all". Alias: `docker ps -a`. |

---

## 🏳️ Flags Reference

| Flag | Full Form | What It Does |
|------|-----------|-------------|
| `-i` | `--interactive` | Keeps STDIN open — allows you to type commands into the container. |
| `-t` | `--tty` | Allocates a pseudo-TTY — gives you a proper terminal prompt. |
| `-it` | — | Combination of `-i` and `-t`. Used together to get a live interactive shell. |
| `-d` | `--detach` | Runs the container in the background (detached mode). Returns the container ID and frees your terminal. |
| `--name <name>` | — | Assigns a custom name to the container instead of a random one (e.g., `inspiring_hopper`). |
| `-a` | `--all` | Used with `docker container ls -a` to show all containers including stopped ones. |

---

## 🧠 Key Concepts

### Image vs Container
```
Image     = Blueprint / Template (read-only, stored on disk)
Container = Running instance of an image (has its own process, filesystem, network)
```

### `docker run` vs `docker exec`
```
docker run   → Creates a NEW container from an image and starts it
docker exec  → Enters an EXISTING running container (no new container created)
```

### Interactive (`-it`) vs Detached (`-d`)
```
-it  → You are INSIDE the terminal, typing commands live
-d   → Container runs SILENTLY in the background, you stay on your host terminal
```

### Why `docker run -d ubuntu` Exits Immediately
Ubuntu's default command is `/bin/bash`. A bash shell without interactive input (`-it`) has nothing to do, so it exits with code 0 immediately. **Fix:** Give it a long-running process:
```bash
docker run -d --name my-ubuntu ubuntu sleep infinity
```

### Exit Codes
```
Exited (0)   → Container ran successfully and exited normally
Exited (1)   → Container encountered an error
Exited (137) → Container was killed (SIGKILL / out of memory)
```

### Container Names
Docker auto-assigns fun random names like `inspiring_hopper`, `blissful_edison`, `great_mclaren` if you don't specify `--name`. Always use `--name` in practice for clarity.

### Permission Denied on `docker.sock`
```bash
# Error: permission denied while trying to connect to the docker API
# Fix: Add your user to the docker group
sudo usermod -aG docker $USER
# Then restart your terminal for it to take effect
```

---

## 🔄 Full Container Lifecycle Summary

```
docker pull ubuntu          # Step 1: Download the image
        │
docker run -d --name        # Step 2: Create & start a container
  my-app ubuntu sleep infinity
        │
docker exec -it             # Step 3: Enter the running container
  my-app /bin/bash
        │
docker stop my-app          # Step 4: Stop the container
        │
docker rm my-app            # Step 5: Delete the container
```

---

> **Labs Covered:** 48 (Install Docker) → 49 (Inspect Images & Containers) → 50 (Run Hello World) → 51 (Interactive & Detached Ubuntu)

