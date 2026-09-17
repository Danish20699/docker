# Lab 60: Demo Bind Mount on Apache Container

## 📌 Objective

Demonstrate **Docker Bind Mounts** using an Apache (`httpd`) web server container. Prove that changes made to files on the host system are reflected inside the running container in real time without restarting or rebuilding the container.

---

## 🧠 What is a Bind Mount?

A **Bind Mount** mounts a specific file or directory from your host machine into a target directory inside the container.

```text
Host System (Windows / WSL)               Container (Apache)
┌───────────────────────────────┐        ┌───────────────────────────────┐
│ C:\Users\danis\Desktop\site\  │   -v   │ /usr/local/apache2/htdocs/    │
│  ├── index.html               │ ═════► │  ├── index.html               │
│  └── styles.css               │        │  └── styles.css               │
└───────────────────────────────┘        └───────────────────────────────┘
  ▲                                        ▲
  │ Edit on host in VS Code                │ Container serves updated
  └────────────────────────────────────────┴─── files instantly!
```

### Key Properties of Bind Mounts:
- **Direct file access**: No file copying occurs (`docker cp` is not required).
- **Instant updates**: Saving a file in your IDE updates the webpage on reload.
- **Bi-directional**: Changes made inside the container can also modify files on the host (unless mounted read-only `:ro`).

---

## ▶️ Hands-On Execution Steps

### Step 1: Create a Website Directory on Host

Create a project directory and an initial `index.html` file on the host machine:

```bash
mkdir -p ~/apache-bind-demo
cat << 'EOF' > ~/apache-bind-demo/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Bind Mount Demo</title>
    <style>
        body { font-family: sans-serif; background: #0f172a; color: #f8fafc; text-align: center; padding-top: 50px; }
        .card { background: #1e293b; padding: 20px; border-radius: 8px; display: inline-block; }
    </style>
</head>
<body>
    <div class="card">
        <h1>🚀 Apache Bind Mount Demo</h1>
        <p>Version 1.0 — Initial Deployment</p>
    </div>
</body>
</html>
EOF
```

---

### Step 2: Run Apache Container with Bind Mount

Mount the directory to Apache's document root (`/usr/local/apache2/htdocs/`):

```bash
docker run -d \
  --name apache-bind-demo \
  -p 8080:80 \
  -v ~/apache-bind-demo:/usr/local/apache2/htdocs/ \
  httpd:alpine
```

Verify container is running:
```bash
docker container ls
```

---

### Step 3: Test Initial Webpage in Browser

1. Open your browser.
2. Navigate to: `http://localhost:8080`
3. Result: Displays **"Version 1.0 — Initial Deployment"**.

---

### Step 4: The Hot-Reload Test (Edit Host File Directly)

Without stopping or restarting the container, edit `index.html` on the host:

```bash
sed -i 's/Version 1.0 — Initial Deployment/Version 2.0 — Live Hot-Reload Successful!/g' ~/apache-bind-demo/index.html
```

---

### Step 5: Verify Live Update in Browser

1. Return to the browser tab (`http://localhost:8080`).
2. Refresh the page (`Ctrl + R` or `Ctrl + Shift + R`).
3. Result: The page immediately displays **"Version 2.0 — Live Hot-Reload Successful!"** without any container downtime!

---

### Step 6: (Optional) Read-Only Bind Mount

In production, to prevent the container from accidentally tampering with host files, append `:ro` to enforce read-only access:

```bash
docker run -d \
  --name apache-readonly \
  -p 8081:80 \
  -v ~/apache-bind-demo:/usr/local/apache2/htdocs/:ro \
  httpd:alpine
```

---

## 🧹 Cleanup

```bash
docker rm -f apache-bind-demo
rm -rf ~/apache-bind-demo
```

---

## 📝 Key Takeaways

1. **Bind mounts are ideal for local development**: You can edit source code in VS Code / Antigravity IDE and see changes instantaneously inside the container.
2. **Path specification**: Always use absolute paths or environment variables (`$(pwd)` / `~/`) when specifying the host source directory.
3. **Permissions**: Ensure container processes have read permissions on host files.
