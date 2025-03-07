# **ELT Pipeline with PostgreSQL and Docker**

## **📌 Overview**
This ELT (Extract, Load, Transform) pipeline automates data transfer between two PostgreSQL databases using **Docker Compose**. It extracts data from `source_postgres`, saves it to an SQL file (`data_dump.sql`), and loads it into `destination_postgres` using PostgreSQL command-line tools.

---

## **📂 Project Structure**
```
elt/
│-- source_db_init/
│   ├── init.sql          # Pre-populates `source_postgres` with tables and data
│-- elt_script.py         # Python script to handle ELT process
│-- Dockerfile            # Builds the ELT container
│-- docker-compose.yml    # Defines the services (PostgreSQL & ELT script)
│-- README.md             # Documentation
```

---

## **⚡ Technologies Used**
- **PostgreSQL 17.4**
- **Docker & Docker Compose**
- **Python 3.8**
- **PostgreSQL CLI Tools (`pg_dump`, `psql`)**

---

## **🚀 Getting Started**

### **1️⃣ Clone the Repository**
```bash
git clone https://github.com/I-Akash-Sharma/elt/
cd elt
```

### **2️⃣ Build and Start the Containers**
```bash
docker compose up -d --build
```
This will:
- Start `source_postgres` (Port **5433**)
- Start `destination_postgres` (Port **5434**)
- Run `elt_script.py` to migrate data

### **3️⃣ Verify Running Containers**
```bash
docker ps
```
You should see:
```
source_postgres   Up (Port 5433)
destination_postgres   Up (Port 5434)
elt_script   Exited (successful completion)
```

### **4️⃣ Check Data in Destination Database**
Connect to `destination_postgres`:
```bash
psql -h localhost -p 5434 -U postgres -d destination_db
```
Run:
```sql
SELECT * FROM users;
SELECT * FROM films;
```
✅ You should see data transferred from `source_db`!

---

## **🛠 How It Works**
### **1️⃣ Database Initialization (`source_db_init/init.sql`)**
- Creates **tables** (`users`, `films`, `film_category`, `actors`, `film_actors`)
- Inserts **sample data** into `source_postgres`

### **2️⃣ ELT Script (`elt_script.py`)**
- **Waits for `source_postgres` to be ready** using `pg_isready`
- **Dumps `source_db` into `data_dump.sql`** using `pg_dump`
- **Loads the dump into `destination_db`** using `psql`

### **3️⃣ Docker Compose (`docker-compose.yml`)**
- Defines **two PostgreSQL services** (`source_postgres`, `destination_postgres`)
- Runs the **ELT script in a Python container**
- Connects all services via **Docker Network (`elt_network`)**

---

## **📄 Configuration Details**
### **Environment Variables (Set in `docker-compose.yml`)**
| Variable         | `source_postgres` | `destination_postgres` |
|-----------------|------------------|-----------------------|
| `POSTGRES_DB`   | source_db        | destination_db       |
| `POSTGRES_USER` | postgres         | postgres             |
| `POSTGRES_PASSWORD` | secret       | secret               |

---

## **📌 Useful Commands**
### **🔄 Restart the Pipeline**
```bash
docker compose down --volumes
```
```bash
docker compose up -d --build
```

### **📜 Check Logs**
```bash
docker logs <container_id>
```

### **🛑 Stop and Remove Containers**
```bash
docker compose down
```

---

## **🎯 Future Improvements**
✅ Implement **data transformations** before loading into `destination_postgres`
✅ Use **environment variables** for database credentials
✅ Add **logging and monitoring** for ELT process

---

## **📢 Support**
For any issues, feel free to open a GitHub **issue** or reach out! 🚀

