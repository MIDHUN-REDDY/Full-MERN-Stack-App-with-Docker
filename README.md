# Full-MERN-Stack-App-with-Docker
# 🚀 Full MERN Stack App with Docker & Nginx Reverse Proxy

This project demonstrates how to build and run a **MERN stack application** (MongoDB, Express.js, React, Node.js) completely using **Docker**.  
It includes a reverse proxy setup with **Nginx**, database admin UI using **Mongo Express**, and isolated Docker networking.

---

## 📦 Step 1: Initial Docker Setup

```bash
docker images           # Show available Docker images
docker ps -a            # Show all containers (even stopped ones)
docker network ls       # List existing Docker networks
docker volume ls        # List existing volumes
```
👉 These commands help verify your system's Docker state before starting.

# 📁 Step 2: Clone the Project
```bash
mkdir docker
cd docker
git clone https://github.com/narayanacharan/mern_docker_demo.git
cd mern_docker_demo/
ls -ltr
docker images
```
👉 This clones the MERN project and prepares the working directory.

# 🌐 Step 3: Create Custom Docker Network and Volume
```bash
docker network create library-mern-api     # Custom Docker network
docker network ls                          # Verify network created

docker volume create mongodb-data          # Persistent volume for MongoDB
docker volume ls                           # Check if volume is created
```
👉 Containers inside the same network can communicate. Volumes persist data even if the container is deleted.

# 🐳 Step 4: Run MongoDB and Mongo Express Containers
```bash
# MongoDB
docker run -dit -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  -e PWD=/ \
  -v mongodb-data:/data/db \
  --name mern_library_nginx_mongodb_1 \
  --net library-mern-api \
  mongo

# Mongo Express
docker run -dit -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  --net library-mern-api \
  --name mern_library_nginx_mongo-express_1 \
  -e ME_CONFIG_MONGODB_SERVER=mern_library_nginx_mongodb_1 \
  -e ME_CONFIG_BASICAUTH_USERNAME=admin \
  -e ME_CONFIG_BASICAUTH_PASSWORD=admin123456 \
  mongo-express

docker ps     # Verify both containers are running
```
👉 Mongo Express is a web-based MongoDB admin tool.
🔐 Access it at: http://localhost:8081
Login: username: admin / passwor: admin123456

# 🏗️ Step 5: Build the Backend (Express API)
```bash
cd server/
ls
docker build -t mern_library_nginx_library-api .
docker images
cd ..
```
👉 This builds the Node.js API Docker image with a custom tag.

# 💻 Step 6: Build the Frontend (React App)
```bash
cd client/
ls
docker build -t mern_library_nginx_client .
docker images
cd ..
```
👉 This builds the React frontend Docker image.

# 🌐 Step 7: Build the Nginx Reverse Proxy
```bash
cd nginx/
ls
docker build -t mern_library_nginx_nginx .
docker images
cd ..
```
👉 Nginx will serve both the frontend and backend via port 8080.

# 🚀 Step 8: Run the Backend Container
```bash
cd server
docker run -dit -p 5000:5000 `
  -e MONGO_URI=mongodb://admin:password@mern_library_nginx_mongodb_1 `
  -e NODE_ENV=development `
  -e PWD=/app `
  -v "${PWD}:/app" `
  -v "${PWD}\node_modules:/app/node_modules" `
  --net library-mern-api `
  --name library_mern_nginx `
  mern_library_nginx_library-api
cd ..
```
👉 The backend connects to MongoDB inside the Docker network.

# 🖥️ Step 9: Run the Frontend Container
```bash
cd client
docker run -d `
  -v "${PWD}:/app" `
  -v "${PWD}\node_modules:/app/node_modules" `
  --net library-mern-api `
  --name library_mern_frontend `
  mern_library_nginx_client
cd ..
```
👉 The React app will be served through Nginx.

# 🌐 Step 10: Run the Nginx Container
```bash
docker run -d -p 8080:80 `
  --name mern_library_nginx_nginx_1 `
  --net library-mern-api `
  mern_library_nginx_nginx

docker ps

```
👉 Nginx is now serving your MERN app via:
🔗 http://localhost:8080

# 📌 Final Setup Summary

| Component        | Port  | URL                       | Notes                  |
|------------------|-------|---------------------------|-------------------------|
| MongoDB          | 27017 | Internal Only             | Backend connects to this |
| Mongo Express    | 8081  | http://localhost:8081     | MongoDB Admin UI        |
| Node.js Backend  | 5000  | Internal via Nginx        | Serves APIs             |
| React Frontend   | N/A   | Served via Nginx          | Built React SPA         |
| Nginx            | 8080  | http://localhost:8080     | Public entrypoint       |

# 🧼 Bonus: Cleanup Commands
```bash
# Stop all containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove all volumes
docker volume prune

# Remove all networks
docker network prune
```
⚠️ Use with caution — this clears everything from your local Docker.

# 🙌 Author
Made by Kakanuru Midhun Reddy
GitHub: MIDHUN-REDDY
