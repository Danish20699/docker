# Lab 61: Run Portfolio Website Using Volume / Bind Mounts Containers

## 📌 Objective

Deploy a production-grade personal portfolio website containerized with Apache (`httpd`), utilizing **Docker Bind Mounts** to mount compiled web assets directly from the local host repository into the container's web root with port forwarding.

---

## 🧠 Architecture Overview

```text
Host System (Windows / WSL)
Desktop/Devops/my_portfolio_final/dist/
├── index.html
├── resume.html
├── assets/
└── images/
       │
       │ -v /mnt/c/Users/danis/Desktop/Devops/my_portfolio_final/dist:/usr/local/apache2/htdocs/
       ▼
┌─────────────────────────────────────────────────────────────┐
│                 Apache Container (portfolio-prod)           │
│                 Image: httpd:alpine                         │
│                 Port: 80 (Internal)                         │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               │ -p 8080:80
                               ▼
                    Browser: http://localhost:8080
```

---

## ▶️ Implementation Steps

### Step 1: Locate Static Production Build on Host

Ensure your production distribution directory exists on the host machine:

* **Windows Path**: `C:\Users\danis\Desktop\Devops\my_portfolio_final\dist`
* **WSL Path**: `/mnt/c/Users/danis/Desktop/Devops/my_portfolio_final/dist`

Directory contents:
- `index.html` (Main portfolio landing page)
- `resume.html` (Interactive resume page)
- `assets/` (Compiled JavaScript & CSS bundles)
- `images/` (Headshot, icons, and project banners)

---

### Step 2: Remove Stale Containers

Free port 8080 by stopping any old containers:

```bash
docker rm -f my-portfolio portfolio-web apache-test 2>/dev/null || true
```

---

### Step 3: Run Portfolio Container with Bind Mount and Port Forwarding

```bash
docker run -d \
  --name portfolio-app \
  -p 8080:80 \
  -v /mnt/c/Users/danis/Desktop/Devops/my_portfolio_final/dist:/usr/local/apache2/htdocs/ \
  httpd:alpine
```

**Flag Explanations:**
- `-d`: Runs detached in the background.
- `--name portfolio-app`: Assigns an explicit identifiable container name.
- `-p 8080:80`: Exposes Apache port 80 on host port 8080.
- `-v /mnt/c/.../dist:/usr/local/apache2/htdocs/`: Live bind-mounts your website files.

---

### Step 4: Verify Container Health & Status

```bash
docker container ls
```

**Output:**
```text
CONTAINER ID   IMAGE          COMMAND              STATUS         PORTS                  NAMES
12914dceee51   httpd:alpine   "httpd-foreground"   Up 12 seconds  0.0.0.0:8080->80/tcp   portfolio-app
```

---

### Step 5: Verify Files Mounted Inside Container

```bash
docker exec -it portfolio-app ls -la /usr/local/apache2/htdocs/
```

**Output:**
```text
total 48
-rwxrwxrwx 1 1000 1000  9856 Aug 18 14:46 apple-touch-icon.png
drwxrwxrwx 1 1000 1000  4096 Aug 22 05:20 assets
-rwxrwxrwx 1 1000 1000  1628 Aug 18 14:46 favicon-32.png
drwxrwxrwx 1 1000 1000  4096 Aug 22 05:20 gallery
drwxrwxrwx 1 1000 1000  4096 Aug 22 05:20 images
-rwxrwxrwx 1 1000 1000  2636 Aug 22 05:20 index.html
-rwxrwxrwx 1 1000 1000 14908 Aug 22 05:19 resume.html
-rwxrwxrwx 1 1000 1000   168 Aug 22 05:19 robots.txt
-rwxrwxrwx 1 1000 1000  2310 Aug 22 05:19 sitemap.xml
drwxrwxrwx 1 1000 1000  4096 Aug 22 05:20 video
drwxrwxrwx 1 1000 1000  4096 Aug 22 05:20 work
```

---

### Step 6: Access in Browser

1. Open **http://localhost:8080** in your browser.
2. The portfolio website renders with full fidelity, styling, images, and responsiveness!

### 📸 Screenshot — Portfolio Live via Docker Bind Mount

![Portfolio Live in Docker](assets/53-portfolio-live-in-docker.png)

---

## 🧹 Cleanup

```bash
docker rm -f portfolio-app
```

---

## 📝 Key Takeaways

1. Bind mounting production distributions (`dist/` or `build/`) allows instantaneous testing of build artifacts in an isolated container environment identical to production.
2. No image rebuilds or image layers are created when content updates; files are streamed directly from host storage.
3. Combining `-p` (port forwarding) with `-v` (bind mount) provides a complete local DevOps workflow for web applications.
