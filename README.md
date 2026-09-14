# docker
Hands-on Docker containerization labs covering container lifecycle, image building with Dockerfiles, volume persistence, multi-container networking, and production orchestration.
# 🐳 Docker Containerization & DevOps Labs

![Docker](https://img.shields.io/badge/Docker-26.0%2B-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu%2024.04-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Containers](https://img.shields.io/badge/Containers-OCI%20Standard-blue?style=for-the-badge&logo=linuxcontainers&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active%20Learning-success?style=for-the-badge)

A comprehensive, hands-on collection of **Docker containerization labs and practical exercises**—progressing from container fundamentals and Linux isolation primitives to custom multi-stage image builds, volume persistence, and multi-container application networking.

---

## 📌 Table of Contents

- [Overview & Architecture](#-overview--architecture)
- [Repository Structure](#-repository-structure)
- [Core Docker Primitives](#-core-docker-primitives)
- [Hands-on Lab Curriculum](#-hands-on-lab-curriculum)
- [Essential Docker Commands Reference](#-essential-docker-commands-reference)
- [Author & Program](#-author--program)

---

## 🏗️ Overview & Architecture

Docker revolutionized modern DevOps by replacing heavy hardware virtualization with lightweight, kernel-level process isolation.

```text
┌─────────────────────────────────────────────────────────┐
│                     HOST HARDWARE                       │
│                   (CPU / RAM / SSD)                     │
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
│ Web / Apache  │     │ Database / PG │     │ Cache / Redis │
└───────────────┘     └───────────────┘     └───────────────┘
