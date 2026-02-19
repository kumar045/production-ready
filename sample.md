Below is a **production-ready Docker setup** for:

* ✅ **Next.js frontend**
* ✅ **FastAPI backend**
* ✅ Docker Compose for deployment
* ✅ Separate Dockerfiles for both
* ✅ Clean networking between services

This assumes the structure:

```
project-root/
│
├── frontend/        (Next.js app)
├── backend/         (FastAPI app)
└── docker-compose.yml
```

---

# 🐳 docker-compose.yml

```yaml
version: "3.9"

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: fastapi_backend
    restart: always
    env_file:
      - ./backend/.env
    ports:
      - "8000:8000"
    networks:
      - app_network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: nextjs_frontend
    restart: always
    env_file:
      - ./frontend/.env
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

---

# 🚀 FastAPI Backend

## 📂 backend/Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y build-essential

# Copy requirements first for caching
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

# Copy app
COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 📂 backend/requirements.txt

```txt
fastapi
uvicorn[standard]
python-dotenv
```

---

## 📂 backend/main.py

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "FastAPI backend running"}
```

---

# 🌐 Next.js Frontend (Production Build)

## 📂 frontend/Dockerfile

```dockerfile
# ----------- Build Stage -----------
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

# ----------- Production Stage -----------
FROM node:20-alpine

WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app ./

EXPOSE 3000

CMD ["npm", "start"]
```

---

## 📂 frontend/package.json (Important)

Make sure you have:

```json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start"
}
```

---

# 🔗 Connecting Frontend to Backend

Inside your **Next.js frontend**, call backend like this:

```js
const response = await fetch("http://backend:8000/");
```

⚠ Important:
Use `backend` as hostname (not localhost). Docker service name acts as internal DNS.

---

# 🚀 Run Everything

From project root:

```bash
docker-compose up -d --build
```

Access:

* Frontend → [http://your-server-ip:3000](http://your-server-ip:3000)
* Backend → [http://your-server-ip:8000](http://your-server-ip:8000)

---

# 🏭 Production Recommendation (With Reverse Proxy)

In production, you should:

* Remove exposed ports
* Use Nginx or Traefik
* Use HTTPS with SSL (Let's Encrypt)
* Possibly add PostgreSQL & Redis
