

```bash
.
├── docker-compose.yml
├── .env                # create this (example below)
└── api
    ├── Dockerfile
    ├── requirements.txt
    └── main.py
```

`docker-compose.yml`  
```text
services:
  db:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u root -p$${MYSQL_ROOT_PASSWORD} --silent"]
      interval: 5s
      timeout: 3s
      retries: 20

  api:
    build: ./api
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_HOST: db
      DB_PORT: 3306
      DB_NAME: ${MYSQL_DATABASE}
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "8000:8000"
    volumes:
      - ./api:/app
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload

volumes:
  db_data:
```

`.env`
```text
MYSQL_ROOT_PASSWORD=devrootpw
MYSQL_DATABASE=appdb
MYSQL_USER=appuser
MYSQL_PASSWORD=apppass
```

`/api/Dockerfile`  
```text
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# copy app code last (for faster rebuilds while editing requirements)
COPY . .

EXPOSE 8000

```


`requirements.txt`  
```text
fastapi
uvicorn[standard]
sqlalchemy>=2.0
pymysql

```

`/api/main.py`  
```text
import os
from fastapi import FastAPI
from sqlalchemy import create_engine, text

app = FastAPI(title="FastAPI + MySQL Demo")

# Build a SQLAlchemy engine (optional—/hi doesn't require DB;
# this just proves the DB container is reachable)
DB_USER = os.getenv("DB_USER", "appuser")
DB_PASSWORD = os.getenv("DB_PASSWORD", "apppass")
DB_HOST = os.getenv("DB_HOST", "db")  # service name from docker-compose
DB_PORT = int(os.getenv("DB_PORT", "3306"))
DB_NAME = os.getenv("DB_NAME", "appdb")

DATABASE_URL = f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
engine = create_engine(DATABASE_URL, pool_pre_ping=True, future=True)

@app.get("/hi")
def hi():
    return {"message": "hi 👋"}

@app.get("/db-ping")
def db_ping():
    # Quick check to see if the DB is accessible
    with engine.connect() as conn:
        result = conn.execute(text("SELECT 1 AS ok")).mappings().first()
    return {"db_ok": bool(result and result["ok"] == 1)}

```

`Run`  
docker compose up --build  
