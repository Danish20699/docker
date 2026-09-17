# Lab 59: Demo Docker Volume on PostgreSQL Container

## 📌 Objective

Demonstrate database data persistence using Docker **Named Volumes** with a PostgreSQL container. Verify that database tables and records survive even when the container is forcefully deleted and recreated.

---

## 🧠 Why Volumes are Critical for Databases

When running relational databases like PostgreSQL in Docker:
- PostgreSQL stores its cluster databases at `/var/lib/postgresql/data` inside the container.
- Without a volume, running `docker rm` will **erase the entire database**.
- By mounting a Docker Named Volume to `/var/lib/postgresql/data`, all tables, indexes, and write-ahead logs (WAL) are safely persisted on the host disk.

```text
┌─────────────────────────────────┐
│     PostgreSQL Container 1      │
│      (psql-test-container)      │
│    writes to /var/lib/postgresql│
└───────────────┬─────────────────┘
                │ -v psql_volume_data:/var/lib/postgresql/data
                ▼
      ┌──────────────────┐
      │   DOCKER VOLUME  │ ◄─── Data persists here safely!
      │ (psql_volume_data│
      └─────────┬────────┘
                ▲
                │ Re-attached via -v psql_volume_data:...
┌───────────────┴─────────────────┐
│     PostgreSQL Container 2      │
│      (Brand New Container)      │
│   reads existing tables instantly│
└─────────────────────────────────┘
```

---

## ▶️ Hands-On Execution Steps

### Step 1: Create a Dedicated Docker Named Volume

```bash
docker volume create psql_volume_data
```

Verify the volume:
```bash
docker volume ls
```

---

### Step 2: Run PostgreSQL Container Attached to Volume

```bash
docker run -d \
  --name psql-primary \
  -e POSTGRES_PASSWORD=adminpassword \
  -e POSTGRES_DB=collegedb \
  -v psql_volume_data:/var/lib/postgresql/data \
  postgres:16-alpine
```

Verify container is running:
```bash
docker container ls
```

---

### Step 3: Connect and Insert Records into PostgreSQL

Access the interactive PostgreSQL CLI (`psql`) inside the running container:

```bash
docker exec -it psql-primary psql -U postgres -d collegedb
```

Inside the `psql` prompt, create a table and insert a record:

```sql
-- Create table
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    course VARCHAR(50)
);

-- Insert sample records
INSERT INTO students (name, course) VALUES ('Danish Nazir', 'DevOps & Cloud');
INSERT INTO students (name, course) VALUES ('Ayaan', 'Full Stack Engineering');

-- Verify records exist
SELECT * FROM students;
```

**Output:**
```text
 id |     name     |         course         
----+--------------+------------------------
  1 | Danish Nazir | DevOps & Cloud
  2 | Ayaan        | Full Stack Engineering
(2 rows)
```

Exit the PostgreSQL CLI:
```sql
\q
```

---

### Step 4: The Destruction Test (Delete the Container)

Force stop and permanently delete the container:

```bash
docker rm -f psql-primary
```

Verify that the container is completely deleted:
```bash
docker container ls -a
```
`psql-primary` is gone.

---

### Step 5: Launch a BRAND NEW Container with the SAME Volume

Start a new container named `psql-recovery` mounting the original volume:

```bash
docker run -d \
  --name psql-recovery \
  -e POSTGRES_PASSWORD=adminpassword \
  -v psql_volume_data:/var/lib/postgresql/data \
  postgres:16-alpine
```

---

### Step 6: Verify Data Persistence in the New Container

Connect to the new `psql-recovery` container:

```bash
docker exec -it psql-recovery psql -U postgres -d collegedb -c "SELECT * FROM students;"
```

**Output:**
```text
 id |     name     |         course         
----+--------------+------------------------
  1 | Danish Nazir | DevOps & Cloud
  2 | Ayaan        | Full Stack Engineering
(2 rows)
```

✅ **Data persistence verified!** All records were preserved across container destruction.

---

## 🧹 Cleanup

```bash
# Stop and delete container
docker rm -f psql-recovery

# Remove volume
docker volume rm psql_volume_data
```

---

## 📝 Key Takeaways

1. PostgreSQL requires `/var/lib/postgresql/data` to be mounted to a persistent volume for production durability.
2. New containers mounting an existing database volume do not require database re-initialization; they immediately resume serving the existing data files.
3. Named volumes decouple data lifecycle from container lifecycle.
