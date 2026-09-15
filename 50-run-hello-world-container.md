# Lab 50: Run Hello World Container

## 📌 Objective

Run the `hello-world` container to verify Docker is working correctly, and understand the 4-step process Docker follows behind the scenes.

---

## ▶️ Command

```bash
docker run hello-world
```

---

## 📋 Output Breakdown

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Docker explains exactly what happened in 4 steps:

### Docker's 4-Step Process

```
┌──────────────────────────────────────────────────────────────────┐
│  1. Docker client contacted the Docker daemon.                   │
│  2. The Docker daemon pulled the "hello-world" image from        │
│     Docker Hub (amd64).                                          │
│  3. The Docker daemon created a new container from that image    │
│     which runs the executable that produces the output.          │
│  4. The Docker daemon streamed the output to the Docker client,  │
│     which sent it to your terminal.                              │
└──────────────────────────────────────────────────────────────────┘
```

| Step | What Happens |
|------|-------------|
| **Step 1** | Your CLI (`docker` command) sends a request to the Docker daemon (background service) |
| **Step 2** | Daemon checks locally for the image → not found → pulls from **Docker Hub** |
| **Step 3** | Daemon creates a container from the image and runs the binary inside it |
| **Step 4** | The container's output (stdout) is streamed back to your terminal |

---

## 🔎 Verification After Running

### Check running containers:

```bash
docker container ls
```

**Result:** Empty — `hello-world` already finished and exited.

### Check all containers (including stopped):

```bash
docker container ls -a
```

**Result:**

```
CONTAINER ID   IMAGE         COMMAND    CREATED            STATUS                        NAMES
fcfa07473783   hello-world   "/hello"   47 seconds ago     Exited (0) 46 seconds ago     inspiring_hopper
8c2bb890dcae   hello-world   "/hello"   9 hours ago        Exited (0) 9 hours ago        blissful_edison
```

- **`Exited (0)`** = the container ran successfully and exited with code 0 (success).
- Two entries exist because `hello-world` was run twice (once 9 hours ago, once just now).

---

## 📸 Screenshot

![Run Hello World Container](assets/49-docker-images-container-ls-hello-world.png)

---

## 📝 Key Takeaways

- `docker run <image>` is the most fundamental Docker command — it **pulls** (if needed), **creates**, and **starts** a container.
- `hello-world` is a test-only image. It prints a message and immediately exits.
- Exited containers are **not deleted automatically** — they remain until you `docker rm` them.
- Each `docker run` creates a **new container**, even from the same image.

