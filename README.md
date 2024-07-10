# Full-Stack FastAPI and React Template

Welcome to the Full-Stack FastAPI and React template repository. This repository serves as a demo application for interns, showcasing how to set up and run a full-stack application with a FastAPI backend and a ReactJS frontend using ChakraUI.

## Project Structure

The repository is organized into two main directories:

- **frontend**: Contains the ReactJS application.
- **backend**: Contains the FastAPI application and PostgreSQL database integration.

Each directory has its own README file with detailed instructions specific to that part of the application.

## Getting Started

To get started with this template, please follow the instructions in the respective directories:

- [Frontend README](./frontend/README.md)
- [Backend README](./backend/README.md)

## DOCKER FILES
 Create separate docker files for the front-end and back-end. Use the stated requirements in the READme to define the contents in the docker file
 #### Frontend Dockerfile
 ```
# Use the latest official Node.js image as a base
FROM node:latest

# Set the working directory
WORKDIR /app

# Copy the application files
COPY . .

# arg command
ARG VITE_API_URL=${VITE_API_URL}

# Install dependencies
RUN npm install

# Expose the port the development server runs on
EXPOSE 5173

# Run the development server
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

#### Backend Dockerfile
```
# Use the latest official Python image as a base
FROM python:latest

# Install Node.js and npm
RUN apt-get update && apt-get install -y \
    nodejs \
    npm

# Install Poetry using pip
RUN pip install poetry

# Set the working directory
WORKDIR /app

# Copy the application files
COPY . .

# Install dependencies using Poetry
RUN poetry install
ENV PYTHONPATH=/app

# Expose the port FastAPI runs on
EXPOSE 8000

# Run the prestart script and start the server
CMD ["sh", "-c", "poetry run bash ./prestart.sh && poetry run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload"]
```

## ENV files
The next thing is to update the .env files in the frontend and backend directory.
#### .env for the front end
```
VITE_API_URL=http://<your_server_IP>:8000
```
#### .env file for the backend
```
# Domain
# This would be set to the production domain with an env var on deployment
DOMAIN=localhost

# Environment: local, staging, production
ENVIRONMENT=local

PROJECT_NAME="Full Stack FastAPI Project"
STACK_NAME=full-stack-fastapi-project

# Backend
BACKEND_CORS_ORIGINS="http://localhost,http://localhost:5173,https://localhost,https://localhost:5173,http://<server-ip>:5173"
SECRET_KEY=2e209ec70ebda52356e0fd702a3703d7
FIRST_SUPERUSER=devops@hng.tech
FIRST_SUPERUSER_PASSWORD=devops#HNG11
USERS_OPEN_REGISTRATION=True
DATABASE_URLpostgresql://${POSTGRES_USER}:{POSTGRES_PASSWORD}@{POSTGRES_SERVER}:5432/{POSTGRES_DB}
# Emails
SMTP_HOST=
SMTP_USER=
SMTP_PASSWORD=
EMAILS_FROM_EMAIL=info@example.com
SMTP_TLS=True
SMTP_SSL=False
SMTP_PORT=587

# Postgres
POSTGRES_SERVER=db-postgres   # name of your db service
POSTGRES_PORT=5432
POSTGRES_DB=apidb
POSTGRES_USER=app
POSTGRES_PASSWORD=app
```

## COMPOSE.YML FILE
Create a compose.yml file that will build up your entire architecture
```
services:
  backend:
    build:
      context: ./backend
      args:
         - VITE_API_URL=http://<server-ip>:8000
    container_name: fastapi_app
    ports:
      - "8000:8000"
    depends_on:
      - db-postgres
    env_file:
      - ./backend/.env

  frontend:
    build:
      context: ./frontend
    container_name: nodejs_app
    ports:
      - "5173:5173"
    env_file:
      - ./frontend/.env

  db-postgres:
    image: postgres:latest
    container_name: postgres_db
    ports:
      - "5432:5432"
    # volumes:
    #   - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: apidb

  adminer:
    image: adminer
    container_name: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db-postgres

  proxy:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx_proxy_manager
    restart: unless-stopped
    ports:

      - '80:80'    # Public HTTP Port
      - '8090:81'    # Admin Web Port
      - '443:443'  # Public HTTPS Port
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
      DB_PGSQL_HOST: "db"
      DB_PGSQL_PORT: 5432
      DB_PGSQL_USER: "proxy_user"
      DB_PGSQL_PASSWORD: "proxy00123"
      DB_PGSQL_NAME: "proxy_db"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db-postgres
      - backend
      - frontend
      - adminer

volumes:
  postgres_data:
  data:
  letsencrypt:
```
## Build the Application
Build the application by running the command:
```
docker-compose up -d
```

## Create your nginx.conf file
```
location /api {
    proxy_pass http://fastapi_app:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location /docs {
    proxy_pass http://fastapi_app:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location /redoc {
    proxy_pass http://fastapi_app:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

```
## Configure Routing

Configure your routing to pass request from your front end on the nginx proxy server to `/docs`, `/redoc` and `/api` in the backend 


**To understand ore of this in details, read this article** [HASHNODE](https://dhebbydavid.hashnode.dev/deploying-a-three-tier-application-in-docker-containers)
