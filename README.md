# 🐳 Docker Containerization & DevOps Labs

![Docker](https://img.shields.io/badge/Docker-26.0%2B-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu%2024.04-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-HTTPD-D22128?style=for-the-badge&logo=apache&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active%20Labs-success?style=for-the-badge)

A comprehensive, production-oriented repository of **Docker containerization labs, real-world multi-container deployments, and practical infrastructure exercises**—spanning from kernel-level process isolation to custom network topologies, volume persistence, and database backend integration.

---

## 📌 Table of Contents

- [Architecture Overview](#-architecture-overview)
- [Hands-on Lab Curriculum](#-hands-on-lab-curriculum)
- [Repository Structure](#-repository-structure)
- [Core Docker Primitives](#-core-docker-primitives)
- [Essential Docker Commands Reference](#-essential-docker-commands-reference)
- [Author & Program](#-author--program)

---

## 🏗️ Architecture Overview

Docker replaces heavy hypervisor hardware virtualization with lightweight, kernel-level process isolation leveraging Linux **Namespaces** (PID, NET, MNT, IPC, UTS) and **Cgroups** (CPU, memory, I/O limits):

```text
┌─────────────────────────────────────────────────────────┐
│                     HOST HARDWARE                       │
│                   (CPU / RAM / NVMe)                    │
└────────────────────────────┬────────────────────────────┘
                             │
                  [ Linux Host Kernel ]
           (Namespaces: Isolation | Cgroups: Limits)
                             │
                      [ Docker Engine ]
       ┌─────────────────────┼─────────────────────┐
       ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│  Container 1  │     │  Container 2  │     │  Container 3  │
│  PHP 8.2 App  │     │ PostgreSQL 15 │     │ Apache HTTPD  │
│  (Port 8080)  │     │ (Port 5432)   │     │  (Port 80)    │
└───────┬───────┘     └───────┬───────┘     └───────────────┘
        │                     │
        └──────────┬──────────┘
                   ▼
     [ User-Defined Bridge Network ]
          (DNS Name Resolution)
```

---

## 📚 Hands-on Lab Curriculum

| Lab | Title & Topic | Key Concepts Practiced | Status |
| :--- | :--- | :--- | :---: |
| **Lab 48** | [Docker Installation & Setup](48-install-docker.md) | Docker Engine installation, daemon service verification, user group configuration | ✅ Completed |
| **Lab 49** | [Docker Engine Inspection](49-docker-commands-output.md) | `docker version`, `docker info`, system architecture verification | ✅ Completed |
| **Lab 50** | [Hello World Container](50-run-hello-world-container.md) | Image pull mechanism, container initialization, exit codes, process lifecycle | ✅ Completed |
| **Lab 51** | [Ubuntu Interactive Container](51-run-ubuntu-interactive-container.md) | Interactive TTY (`-it`), detached mode (`-d`), process retention with `sleep` | ✅ Completed |
| **Lab 52** | [Apache HTTPD Container](52-run-apache-interactive-container.md) | Web server initialization, default DocumentRoot exploration, container CLI | ✅ Completed |
| **Lab 53** | [Container Lifecycle Management](53-docker-container-lifecycle.md) | `create`, `start`, `stop`, `pause`, `unpause`, `restart`, `kill`, `rm` | ✅ Completed |
| **Lab 54** | [HTTPD Port Forwarding](54-docker-httpd-container-run-with-port-forwarding.md) | Host-to-container port mapping (`-p 8080:80`), NAT translation, curl validation | ✅ Completed |
| **Lab 55** | [VerveTech Custom Image Run](55-run-verventech-website-container-with-port-forwarding.md) | Third-party registry image pull, multi-port allocation, production web serving | ✅ Completed |
| **Lab 56** | [Docker Networking Topologies](56-docker-networking-types-with-commands.md) | Bridge, host, none networks, user-defined bridge DNS resolution, inter-container ping | ✅ Completed |
| **Lab 57** | [Multi-Tier Portfolio Stack](57-run-portfolio-project-using-2-containers.md) | 2-tier application (PHP 8.2 + PostgreSQL 15), custom bridge network, DB driver compile | ✅ Completed |
| **Lab 58** | [Docker Storage Architecture](58-docker-storage.md) | Storage drivers (overlay2), bind mounts vs named volumes, persistent data lifecycle | ✅ Completed |
| **Lab 59** | [PostgreSQL Volume Persistence](59-demo-docker-volume-on-psql-container.md) | Named volume data retention across container deletion and recreation | ✅ Completed |
| **Lab 60** | [Demo Bind Mount on Apache](60-demo-bind-mount-on-apache-container.md) | Host-to-container directory binding, real-time live hot-reloading, mount inspection | ✅ Completed |

---

## 🗂️ Repository Structure

```text
docker/
├── assets/                                     # Terminal verification screenshots
│   ├── lab56-network-inspect.png
│   ├── lab57-psql-db-setup.png
│   ├── lab57-web-browser-verify.png
│   ├── lab58-volume-inspect.png
│   ├── lab59-psql-table-created.png
│   ├── lab59-psql-data-persisted.png
│   ├── lab60-apache-bind-run.png
│   ├── lab60-bind-mount-test-inspect.png
│   └── lab60-browser-live-update.png
├── 48-install-docker.md
├── 49-docker-commands-output.md
├── 50-run-hello-world-container.md
├── 51-run-ubuntu-interactive-container.md
├── 52-run-apache-interactive-container.md
├── 53-docker-container-lifecycle.md
├── 54-docker-httpd-container-run-with-port-forwarding.md
├── 55-run-verventech-website-container-with-port-forwarding.md
├── 56-docker-networking-types-with-commands.md
├── 57-run-portfolio-project-using-2-containers.md
├── 58-docker-storage.md
├── 59-demo-docker-volume-on-psql-container.md
├── 60-demo-bind-mount-on-apache-container.md
└── README.md
```

---

## ⚡ Essential Docker Commands Reference

| Action | Command | Purpose |
| :--- | :--- | :--- |
| **Run Container** | `docker run -d --name web -p 80:80 nginx` | Start detached container with port forwarding |
| **Interactive Terminal** | `docker exec -it <container-id> bash` | Open interactive shell inside running container |
| **List Containers** | `docker ps -a` | List all active and exited containers |
| **View Logs** | `docker logs -f <container-id>` | Stream live standard output / error logs |
| **Create Network** | `docker network create my-net` | Create isolated custom bridge network with DNS |
| **Create Volume** | `docker volume create my-vol` | Allocate managed persistent storage on host |
| **Inspect Object** | `docker inspect <id>` | View low-level JSON configuration and network state |
| **Clean Up** | `docker system prune -af` | Remove unused containers, images, and dangling data |

---

## 👤 Author

- **Engineer**: Danish Nazir
- **Track**: MLOps & DevOps Engineering
- **GitHub**: [@Danish20699](https://github.com/Danish20699)
- **LinkedIn**: [danish-nazir1](https://www.linkedin.com/in/danish-nazir1/)
