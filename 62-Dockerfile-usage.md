# 62 - Dockerfile Usage & Layer Architecture

![Docker](https://img.shields.io/badge/Docker-26.0%2B-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Alpine](https://img.shields.io/badge/Base-Alpine%203.19-0D597F?style=for-the-badge&logo=alpinedinux&logoColor=white)
![Architecture](https://img.shields.io/badge/Build-Immutable%20Layers-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

---

## 📌 Lab Overview & Objectives

In preceding labs, we ran pre-existing images published on Docker Hub (`postgres:15`, `httpd:alpine`, `php:8.2-apache`). In enterprise DevOps environments, engineers must build **custom container images** tailored specifically for their own applications, microservices, and internal dependencies.

This lab covers the fundamentals of **Dockerfile authoring and image layer architecture**:
1. **Understand Dockerfiles:** A Dockerfile is a simple, declarative text recipe containing an ordered series of instructions used to automatically assemble a Docker image.
2. **Master Core Vocabulary:** Gain hands-on mastery over fundamental Dockerfile instructions: `FROM`, `LABEL`, `ENV`, `RUN`, `WORKDIR`, `COPY`, `EXPOSE`, and `CMD`.
3. **Understand Immutable Layers:** Understand how Docker's storage driver creates cached, read-only filesystem layers for each instruction.
4. **Hands-on Execution:** Build a custom lightweight web server image (`my-first-image:1.0`), inspect its immutable layer hierarchy using `docker history`, and launch a live container on port `8082`.

---

## 🏗️ Dockerfile Layered Architecture

Docker images are assembled as a stack of **immutable (read-only) layers**. Each command in a Dockerfile adds a distinct layer to the image stack:

```text
┌────────────────────────────────────────────────────────┐
│  CMD ["lighttpd", "-D", "-f", "lighttpd.conf"]         │ 0 B (Metadata)
├────────────────────────────────────────────────────────┤
│  EXPOSE 80                                             │ 0 B (Metadata)
├────────────────────────────────────────────────────────┤
│  COPY index.html .                                     │ 24.6 kB (Web Assets)
├────────────────────────────────────────────────────────┤
│  WORKDIR /var/www/localhost/htdocs                     │ 4.1 kB (Directory State)
├────────────────────────────────────────────────────────┤
│  RUN apk update && apk add --no-cache lighttpd curl    │ 11.2 MB (Installed Packages)
├────────────────────────────────────────────────────────┤
│  ENV APP_USER=danish | LABEL maintainer=danish         │ 0 B (Environment Config)
├────────────────────────────────────────────────────────┤
│  FROM alpine:3.19 (Base OS Rootfs)                     │ 8.08 MB (Base Layer)
└────────────────────────────────────────────────────────┘
          ▲
          └── Immutable Read-Only Layers (docker history)
```

---

## 📚 Core Dockerfile Vocabulary Reference

| Instruction | Purpose & Execution Context | Real-World Role |
| :--- | :--- | :--- |
| **`FROM`** | Defines the base image to build upon. Must be the first non-comment instruction. | Sets the operating system foundation (e.g. `alpine:3.19`, `ubuntu:22.04`). |
| **`LABEL`** | Adds custom metadata to the image (e.g., maintainer, version, description). | Documents ownership and organizational metadata. |
| **`ENV`** | Defines persistent environment variables baked directly into the image. | Configures application parameters available during build AND runtime. |
| **`RUN`** | Executes commands **during the build process** and commits the result as a new layer. | Installs software packages, compiles code, creates system users. |
| **`WORKDIR`** | Sets the default working directory for subsequent instructions (`RUN`, `CMD`, `COPY`). | Replaces risky `cd` commands with a consistent build context. |
| **`COPY`** | Copies files or directories from the host build context into the container filesystem. | Injects source code, templates, or configuration files into the image. |
| **`EXPOSE`** | Documents the network ports on which the container will listen at runtime. | Acts as internal architectural documentation (does not publish ports itself). |
| **`CMD`** | Defines the default command and arguments executed **when the container boots**. | Starts the main long-running application process (e.g., web server daemon). |

---

## 🛠️ Step-by-Step Hands-on Execution

### Step 1: Create Build Directory & Source HTML File

We create a dedicated workspace and an HTML document containing student identity:

```bash
mkdir -p ~/dockerfile-demo && cd ~/dockerfile-demo

cat << 'EOF' > index.html
<!DOCTYPE html>
<html>
<head><title>Lab 62 - Dockerfile Usage</title></head>
<body style="font-family: Arial; text-align: center; padding-top: 50px; background: #0f172a; color: white;">
    <h1>🐳 Dockerfile Usage &amp; Layers Demo</h1>
    <p>Built from immutable layers using a Dockerfile recipe.</p>
    <p>Student: <b>Danish Nazir</b></p>
</body>
</html>
EOF
```

---

### Step 2: Author the Dockerfile

We write our `Dockerfile` specifying base image, environment variables, dependencies, and entrypoint:

```dockerfile
# 1. Base image layer
FROM alpine:3.19

# 2. Metadata & Environment layer
LABEL maintainer="danish"
ENV APP_USER="danish"

# 3. Package installation layer (RUN)
RUN apk update && apk add --no-cache lighttpd curl

# 4. Working directory
WORKDIR /var/www/localhost/htdocs

# 5. Copy file layer (COPY)
COPY index.html .

# 6. Document port layer (EXPOSE)
EXPOSE 80

# 7. Default runtime command (CMD)
CMD ["lighttpd", "-D", "-f", "/etc/lighttpd/lighttpd.conf"]
```

---

### Step 3: Build Custom Image (`docker build`)

We trigger Docker's BuildKit engine using `docker build`, tagging the image as `my-first-image:1.0`:

```bash
docker build -t my-first-image:1.0 .
```

**Build Output:**
```text
[+] Building 24.4s (9/9) FINISHED                                                                        docker:default
 => [internal] load build definition from Dockerfile                                                               0.2s
 => [internal] load metadata for docker.io/library/alpine:3.19                                                     4.2s
 => [1/4] FROM docker.io/library/alpine:3.19@sha256:6baf43584bcb78f2e5847d1de515f23499913ac9f12bdf834811a3145eb11  2.7s
 => [2/4] RUN apk update && apk add --no-cache lighttpd curl                                                      15.0s
 => [3/4] WORKDIR /var/www/localhost/htdocs                                                                        0.1s
 => [4/4] COPY index.html .                                                                                        0.1s
 => exporting to image                                                                                             1.8s
 => => naming to docker.io/library/my-first-image:1.0                                                              0.0s
```

---

### Step 4: Inspect Image Layers via `docker history`

To verify the layered architecture taught by the instructor, we inspect the exact layer history of our newly built image:

```bash
docker history my-first-image:1.0
```

**Terminal Verification:**
```text
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
935e786bed33   2 seconds ago   CMD ["lighttpd" "-D" "-f" "/etc/lighttpd/lig…   0B        buildkit.dockerfile.v0
<missing>      2 seconds ago   EXPOSE [80/tcp]                                 0B        buildkit.dockerfile.v0
<missing>      2 seconds ago   COPY index.html . # buildkit                    24.6kB    buildkit.dockerfile.v0
<missing>      2 seconds ago   WORKDIR /var/www/localhost/htdocs               4.1kB     buildkit.dockerfile.v0
<missing>      3 seconds ago   RUN /bin/sh -c apk update && apk add --no-ca…   11.2MB    buildkit.dockerfile.v0
<missing>      3 seconds ago   ENV APP_USER=danish                             0B        buildkit.dockerfile.v0
<missing>      3 seconds ago   LABEL maintainer=danish                         0B        buildkit.dockerfile.v0
<missing>      11 months ago   CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      11 months ago   ADD alpine-minirootfs-3.19.9-x86_64.tar.gz /…   8.08MB    buildkit.dockerfile.v0
```

---

### Step 5: Run Container from Custom Image & Verify

We launch a container from `my-first-image:1.0` mapping port `8082` to container port `80`:

```bash
docker run -d --name lab62-test -p 8082:80 my-first-image:1.0
```

**Container ID:**
```text
c434a644c80c381b29611af22e60ae86a295ec4fe347b997d2371305100a2d2e
```

We query the running web application via `curl`:

```bash
curl -s http://localhost:8082 | grep -i "Danish Nazir"
```

**Output:**
```html
    <p>Student: <b>Danish Nazir</b></p>
```

---

## 📊 Key Takeaways & Best Practices

1. **Layer Caching:** Docker caches unchanged layers. If only `index.html` changes, Docker skips re-running `apk add`, resulting in builds that complete in milliseconds.
2. **Layer Optimization:** Combine related commands using `&&` (e.g. `apk update && apk add ...`) to minimize the total number of intermediate image layers.
3. **Difference Between `RUN` and `CMD`:**
   - `RUN` executes **during image build time** to install software or compile code.
   - `CMD` executes **at container runtime** to start the application process.

---

## 👤 Lab Verification & Author

- **DevOps Engineer:** Danish Nazir
- **Track:** MLOps & DevOps Engineering (VerveTech)
- **Built Image:** `my-first-image:1.0`
- **Active Container:** `lab62-test` (Port `8082:80`)
- **Verification Status:** ✅ Complete (Dockerfile created, image built, layers inspected, container verified)
