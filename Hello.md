```
Sure — here’s a clean, professional setup using oracledb, python-dotenv, and a modular structure (db.py, app.py, .env). This is the kind of structure you'd see in real-world projects, and it's production-friendly while staying simple.


---

.env

DB_USER=your_username
DB_PASSWORD=your_password
DB_DSN=your_host:your_port/your_service_name


---

db.py

import oracledb
from dotenv import load_dotenv
import os

load_dotenv()

# Load environment variables
DB_USER = os.getenv("DB_USER")
DB_PASSWORD = os.getenv("DB_PASSWORD")
DB_DSN = os.getenv("DB_DSN")

# Create a connection pool (recommended for performance)
pool = oracledb.create_pool(
    user=DB_USER,
    password=DB_PASSWORD,
    dsn=DB_DSN,
    min=1,
    max=5,
    increment=1,
    encoding="UTF-8"
)

def get_connection():
    return pool.acquire()


---

app.py

from db import get_connection

def fetch_data():
    with get_connection() as conn:
        with conn.cursor() as cursor:
            cursor.execute("SELECT * FROM your_table")
            for row in cursor:
                print(row)

if __name__ == "__main__":
    fetch_data()


---

Install Dependencies

pip install oracledb python-dotenv


---

Let me know if you want:

SQLAlchemy integration

Async version

Error logging

Query parameters for safety


I can layer it up cleanly.


```
