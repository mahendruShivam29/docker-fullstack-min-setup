# This maybe not working try to use Readme2.md

#### Minimum docker and docker compose setup for full stack project  

```bash
fullstack-project/
├── backend/
│   ├── app/
│   │   └── main.py
│   ├── Dockerfile
│   └── requirements.txt
├── db/
│   └── init.sql
├── frontend/
│   ├── public/
│   ├── src/
│   ├── Dockerfile
│   ├── .dockerignore
│   └── package.json
└── docker-compose.yml
```

Services: 3 containers each for backend (fastapi), db (mysql), frontend (react).  

backend:  
write backend/Dockerfile for running the backend.  
In docker-compose.yml file under environment add all the flags like user, password.  

db:  
use base image only.  
In docker-compose.yml file under environment add all the flags like user, password too.  

frontend:  
write frontend/Dockerfile for running the frontend.  

In docker-compose.yml mention network driver as bridge so all 3 containers are connected.  
Add docker managed volume to be used for database persistence.  

`docker-compose.yml`  
```yml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    volumes:
      - ./backend/app:/code/app
    environment:
      - MYSQL_USER=root
      - MYSQL_PASSWORD=root
      - MYSQL_DATABASE=myappdb
      - DB_HOST=db
    depends_on:
      - db
    networks:
      - app-network

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - app-network

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myappdb
      MYSQL_USER: root
      MYSQL_PASSWORD: root
    volumes:
      - db-data:/var/lib/mysql
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    ports:
      - "3306:3306"

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```
