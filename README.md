# Creating a Data Pipeline from Scratch

### **1. Importing Libraries**

```python
import subprocess
import time
```

- **`subprocess`**: This module provides a higher-level interface to working with additional processes. It allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes. It's a powerful tool for running shell commands directly from Python scripts.
- **`time`**: This module provides time-related functions. In this script, it's primarily used for the **`sleep`** function, which introduces a delay.

---

### **2. `wait_for_postgres` Function**

```python
def wait_for_postgres(host, max_retries=5, delay_seconds=5):
    ...
```

This function's purpose is to repeatedly check if a PostgreSQL instance is ready to accept connections.

- **`host`**: The hostname or IP address of the PostgreSQL server.
- **`max_retries`**: This sets a limit on how many times the function will attempt to connect to the database before it gives up. It's a way to prevent infinite loops if the database is not available.
- **`delay_seconds`**: After each failed attempt, the function will wait for this many seconds before trying again.

Inside the function:

- **`pg_isready`**: A command-line utility that comes with PostgreSQL. It checks the connection status of a PostgreSQL database instance. The function uses this utility to determine if the database is ready.
- **`subprocess.run()`**: This method is used to execute the **`pg_isready`** command. If the command fails (i.e., the database isn't ready), a **`CalledProcessError`** exception is raised, which the function catches and handles by waiting and retrying.

---

### **3. Initializing the ELT Process**

```python
if not wait_for_postgres(host="source_postgres"):
    exit(1)
```

Before the main ELT process starts, the script ensures that the source PostgreSQL database is operational. If the database isn't ready after the specified number of retries, the script terminates with an error code (**`exit(1)`**).

---

### **4. Database Configurations**

```python
source_config = {...}
destination_config = {...}
```

These dictionaries store the necessary configurations for both the source and destination PostgreSQL databases:

- **`dbname`**: The name of the database.
- **`user`**: The username to connect as.
- **`password`**: The password used for authentication.
- **`host`**: The server's hostname or IP address. In the context of Docker Compose, you can use the service name as the hostname.

---

### **5. Dumping Data from the Source Database**

```python
dump_command = [...]
```

This section constructs the command to use **`pg_dump`**, a utility for backing up a PostgreSQL database. The command's parameters are:

- **`h`**: Specifies the server's hostname.
- **`U`**: Specifies the username to connect as.
- **`d`**: Specifies the name of the database to connect to.
- **`f`**: Specifies the filename to which the dump will be written.
- **`w`**: Instructs **`pg_dump`** not to prompt for a password.

---

### **6. Loading Data into the Destination Database**

```python
load_command = [...]
```

This section constructs the command to use **`psql`**, a terminal-based front-end to PostgreSQL. The command's parameters are:

- **`h`**: Specifies the server's hostname.
- **`U`**: Specifies the username to connect as.
- **`d`**: Specifies the name of the database to connect to.
- **`a`**: Echoes all input from script to stdout.
- **`f`**: Specifies the filename from which to read and execute commands.

---

### **7. Environment Variables for Authentication**

```python
subprocess_env = dict(PGPASSWORD=source_config['password'])
```

To avoid being prompted for a password when running **`pg_dump`** and **`psql`**, the script sets the **`PGPASSWORD`** environment variable. This variable holds the password for the specified PostgreSQL user.

---

### **8. Executing the Commands**

```python
subprocess.run(dump_command, env=subprocess_env, check=True)
```

The **`subprocess.run()`** method is used to execute the constructed commands. The **`env`** parameter is used to pass environment variables to the command. The **`check=True`** parameter ensures that if the command results in an error, a **`CalledProcessError`** exception is raised.

---

### **Summary:**

This script is a technical representation of an ELT process, specifically tailored for PostgreSQL databases. It ensures the source database is ready, dumps its data, and then loads this data into the destination database. The script leverages PostgreSQL's built-in utilities (**`pg_dump`** and **`psql`**) and Python's **`subprocess`** module to execute shell commands. The use of Docker Compose service names as hostnames indicates that this script is designed to run within a Dockerized environment, where services can reference each other by their service names.
