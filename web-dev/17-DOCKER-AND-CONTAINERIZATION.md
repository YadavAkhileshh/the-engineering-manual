# 17 Docker and Containerization

This guide covers containerization, virtual machines, Docker engines, Dockerfiles, Compose setups, networks, volumes, multi-stage builds, and container security.

## Containerization vs Virtualization

### Definition
Virtualization runs multiple operating systems on a single physical machine using a Hypervisor to allocate virtual hardware. Containerization packages applications and their dependencies, sharing the host OS kernel through a container engine (Docker) for near-zero overhead.

### Real-world Analogy
Virtualization is like building 3 separate houses on a plot of land. Each house has its own plumbing, heating, and foundation (guest OS). Containerization is like renting 3 apartments in a single building: they share the same building pipes and heating infrastructure (host kernel) but remain locked and isolated from each other.

### Code Example
```bash
# Docker has low overhead and boots in milliseconds:
# docker run --name test-container -d nginx

# In contrast, a virtual machine requires booting a full OS image,
# loading kernels, and allocating dedicated RAM (e.g. 2GB) upfront.
```

### Common Interview Questions
- Compare Virtual Machines and Containers regarding resource isolation and boot times.
- What is a Hypervisor and how does its role differ from the Docker container engine?
- Why do containers share the host operating system kernel, and what are the security implications?

### Reference Links
- [Docker: What is a Container?](https://www.docker.com/resources/what-container/)
- [Docker: Containers vs Virtual Machines](https://docs.docker.com/get-started/docker-overview/)

## Docker Fundamentals

### Definition
Docker is an open-source platform that automates container deployment. Key concepts include Images (read-only blueprints), Containers (running image instances), Dockerfile (build instructions), Volumes (persistent storage), and Networks (inter-container communication).

### Real-world Analogy
An Image is a baking recipe printed in a book (read-only, blueprint). A Container is a cake you bake using the recipe (running instance). You can bake 10 cakes from the same recipe sheet. A Volume is a storage box you place next to the cake to hold cookies that don't spoil if you throw the cake away.

### Code Example
```bash
# Build an image from a Dockerfile in the current directory
docker build -t my-app:1.0 .

# Run container with port mapping and volume attachment
docker run -p 3000:3000 -v my-data:/app/data --name app-instance -d my-app:1.0
```

### Common Interview Questions
- Describe the client-server architecture of Docker (Docker Daemon and Docker CLI).
- What is the difference between a Docker Image and a Docker Container?
- How does the Union File System (UnionFS) work in Docker image layers?

### Reference Links
- [Docker Docs: Architecture](https://docs.docker.com/guides/docker-overview/)

## Dockerfile Instructions

### Definition
A Dockerfile is a text document containing instructions to assemble an image. Instructions include FROM (base image), WORKDIR (directory), COPY (copy files), RUN (execute shell commands), EXPOSE (port declaration), CMD, and ENTRYPOINT.

### Real-world Analogy
Think of assembly instructions for a desk. FROM is buying raw timber. WORKDIR is walking into the workshop. COPY is bringing screws from your toolbox. RUN is drilling holes in the timber. CMD is setting the default desk configuration, which the buyer can override if they bring their own drawers.

### Code Example
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Common Interview Questions
- Compare CMD and ENTRYPOINT instructions in a Dockerfile.
- What happens when you override CMD during docker run?
- Why is it recommended to use npm ci instead of npm install inside a Dockerfile RUN instruction?

### Reference Links
- [Docker Docs: Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)

## Docker Image Optimization

### Definition
Docker image optimization reduces file sizes and speeds up build cycles. Strategies include layer caching (ordering instructions from least-to-most changing), using Alpine base images, configuring .dockerignore files, and multi-stage builds.

### Real-world Analogy
Imagine packing a suitcase. Instead of throwing your dirty boots and heavy winter jackets in the bag (fat images), you filter out items you don't need at the beach using a checklist (.dockerignore), pack heavier items first (layer caching), and travel with only lightweight shorts (alpine bases).

### Code Example
```dockerfile
# Optimized Dockerfile using Layer Caching
FROM node:18-alpine

WORKDIR /usr/src/app

# Package files change rarely, copy first to cache node_modules layer
COPY package*.json ./
RUN npm ci

# Source files change frequently, copy last
COPY . .

CMD ["node", "index.js"]
```

### Common Interview Questions
- How does Docker's layer caching mechanism determine if it can reuse a cached layer?
- Why does placing COPY . . before RUN npm install void layer caching optimization?
- What are the risks and benefits of using Alpine Linux as a base image?

### Reference Links
- [Docker Docs: Optimize Images](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Docker Compose

### Definition
Docker Compose is a tool for defining and running multi-container Docker applications. A docker-compose.yml file configures application services, shared networks, environment variables, dependencies, and volumes.

### Real-world Analogy
Instead of directing 5 workers individually using a megaphone, telling the plumber, electrician, and carpenter when to start and what wires to connect (running docker run commands manually), you hand them a master schedule sheet (docker-compose.yml) that coordinates their tasks automatically.

### Code Example
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://db:5432/mydb
  db:
    image: postgres:15-alpine
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret

volumes:
  db_data:
```

### Common Interview Questions
- What does the depends_on field enforce in docker-compose? Does it guarantee database readiness?
- How do you share environment variables between your host machine and a Docker Compose stack?
- How do you scale a service (e.g. web=3) using Docker Compose commands?

### Reference Links
- [Docker Docs: Compose](https://docs.docker.com/compose/)

## Docker Networking

### Definition
Docker Networking manages communication between containers and external networks. Drivers include Bridge (default isolated host network), Host (bypasses network namespace limits), Overlay (cross-host cluster networks), and None (isolated).

### Real-world Analogy
Bridge network is like an office intercom: all phones connected to the office switchboard can dial each other by cubicle number (DNS container name). Host network is like routing calls directly through the main building telephone line. None network is throwing the phone in the trash.

### Code Example
```bash
# Create a custom bridge network
docker network create my-bridge-net

# Run containers on the shared network (they can ping each other using container names)
docker run --name db-service --network my-bridge-net -d postgres
docker run --name app-service --network my-bridge-net -p 3000:3000 -d node-app
```

### Common Interview Questions
- How does automatic DNS resolution work within custom Docker bridge networks?
- Compare host networking driver and bridge networking driver.
- What is the overlay network driver and when is it used (e.g. Swarm/Kubernetes)?

### Reference Links
- [Docker Docs: Network Overview](https://docs.docker.com/network/)

## Docker Volumes and Data Persistence

### Definition
Docker volumes map host directories to container filesystems, preventing data loss when containers are destroyed. Types include Named Volumes (managed by Docker), Anonymous Volumes, and Bind Mounts (direct host directory links).

### Real-world Analogy
A container is like a rental car. If you buy groceries and put them in the glove compartment, they disappear when you return the car. A Volume is like attaching a personal trailer to the car towbar: you pack your groceries in the trailer, return the car, and attach the trailer to your next car.

### Code Example
```bash
# Named Volume attachment
docker run -v pg_data:/var/lib/postgresql/data postgres

# Bind Mount attachment (maps local folder directly to container)
docker run -v C:\Projects\webdev\src:/app/src node-app
```

### Common Interview Questions
- What is the difference between a Bind Mount and a Named Volume?
- Where does Docker store named volumes on the host system filesystem?
- How do you clean up unused dangling volumes to reclaim disk space?

### Reference Links
- [Docker Docs: Volumes](https://docs.docker.com/storage/volumes/)

## Multi-stage Builds

### Definition
Multi-stage builds use multiple FROM instructions in a single Dockerfile. Developers compile resources and install build tools in early stages, then copy completed artifacts into lightweight runtime stages, producing compact production images.

### Real-world Analogy
Imagine baking a cake. Stage 1 is the kitchen where you use mixers, flour sacks, bowls, and ovens to bake the cake. Stage 2 is placing the completed cake on a clean serving box and carrying it to the table. You don't bring the flour bags, mixers, or dirty ovens to the table.

### Code Example
```dockerfile
# Stage 1: Build environment
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build # Compiles TypeScript/bundles frontend

# Stage 2: Runtime environment
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --only=production
# Copy built artifacts from the builder stage
COPY --from=builder /app/dist ./dist

CMD ["node", "dist/server.js"]
```

### Common Interview Questions
- Why do multi-stage builds significantly reduce production Docker image sizes?
- Explain the role of the AS keyword in Dockerfile FROM instructions.
- How do you copy files from a third-party image directly without building it yourself?

### Reference Links
- [Docker Docs: Multi-stage Builds](https://docs.docker.com/build/building/multi-stage/)

## Container Security Best Practices

### Definition
Container security mitigates threat vectors. Practices include running processes as non-root users, scanning images for vulnerabilities, restricting write access (read-only rootfs), managing secrets securely, and setting resource CPU/memory limits.

### Real-world Analogy
Container security is like managing a laboratory. You don't grant visitors master keycards (run as root). You check their badges for clean records (vulnerability scans), lock the cupboards so they can only read folders but not edit them (read-only filesystem), and restrict them from using all the lab electricity (resource limits).

### Code Example
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

# Run as non-root node user (provided by node alpine image)
USER node

EXPOSE 3000
CMD ["node", "server.js"]
```

### Common Interview Questions
- Why is running a container process as root considered a critical security vulnerability?
- How do you enforce resource limits (CPU and memory allocation) on running containers?
- Why should database passwords and API keys never be baked into Dockerfile ENV instructions?

### Reference Links
- [Docker Docs: Security](https://docs.docker.com/engine/security/)
- [OWASP: Container Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

## Scenario-based Interview Questions for Docker

### Scenario 1
Your React app's Docker image is 1.2GB in size, and deploying it takes 5 minutes on CI. The build process installs node_modules, runs webpack compile, and copies everything into the image. How do you optimize the image size and build speed?

*Expected Approach:*
1. Implement a Multi-Stage Build: Use node-alpine as a builder stage to install devDependencies and run npm run build.
2. In the final stage, use a lightweight web server image like nginx:alpine.
3. Copy only the compiled /dist folder from the builder stage into the Nginx public folder, reducing the image size from 1.2GB to under 50MB and speeding up CI deployment.

### Scenario 2
You run a PostgreSQL container using Docker. When you stop the container (docker stop db-container) and start it back up, your tables are intact. However, when you run docker rm db-container and spawn a new container from the same image, all database records are lost. Why is this happening and how do you fix it?

*Expected Approach:*
1. Explain that containers are ephemeral: any files written inside the container layers are destroyed when the container is removed (docker rm).
2. Fix this by attaching a Named Docker Volume or a Bind Mount to the PostgreSQL storage directory (e.g. -v postgres_data:/var/lib/postgresql/data).
3. This persists the database files on the host machine disk separate from the container lifespan, keeping data safe even if the container is deleted and recreated.
