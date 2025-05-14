

# Docker Concepts Advanced Cheat Sheet

This cheat sheet summarizes the essential ideas and commands that help you understand how Docker works and how to use it effectively.

---

## 📦 Core Docker Concepts

### 1. **Image**
- A snapshot/blueprint of an application and its environment.
- Built from a Dockerfile.
- Immutable (doesn’t change after it’s built).
- **Example:** `node:18`, `postgres:16`, or your custom `mycompany/base-server`.
- Occupies **disk space only** — not loaded into RAM unless a container is started.
  - Stored on disk in the Docker engine’s storage backend (e.g., `/var/lib/docker` on Linux).
  - On Windows/macOS, Docker runs in a lightweight Linux VM, and images are stored inside the VM’s disk volume.
- Removed by `docker system prune` **if** unused (i.e., not referenced by any container).

### 2. **Container**
- A running instance of an image.
- Isolated from the host and other containers.
- Has its own filesystem, processes, and network.
- Multiple containers can be created from the same image.
- Runs in **RAM** while active — loaded from the image and runs processes in memory.
  - When started, the image is used as a base and a temporary *writable layer* is added on top (stored on disk).
  - The container’s layered filesystem is stored alongside the image data in Docker’s internal storage (e.g., `/var/lib/docker/containers` on Linux).
- When stopped, the container's **memory is freed**, but its **writable layer on disk remains** unless the container is removed.
- For **persistent data**, containers use **volumes** (see section on Volumes) — volumes are not tied to the image and survive container deletion.
- On Windows/macOS, the container runs inside a lightweight **Linux VM**. Both disk and memory usage happen within that VM.
- Stopped containers are deleted by `docker system prune` (unless explicitly kept).

### 3. **Dockerfile**
- A script with instructions to build a Docker image.
- Defines base image, files to copy, commands to run, etc.

### 4. **Docker Compose**
- Tool for defining and running multi-container Docker applications.
- Uses a `docker-compose.yml` file to specify services, networks, and volumes.

### 5. **Registry**
- Storage and distribution system for Docker images.
- **Examples:** Docker Hub, GitHub Container Registry, AWS ECR.

---

## 🗄️ Persistent Storage

### 6. **Volume**
- A persistent storage mechanism managed by Docker.
- Used by containers to store data that should survive container restarts or deletion.
- Volumes are **not part of the image** — they are **mounted separately** and hold external data (like databases or uploads).
- Stored on **disk**, typically under `/var/lib/docker/volumes/` on Linux.
- On Windows/macOS, volumes are stored inside the Docker Linux VM’s disk volume.
- Commonly used for databases, user uploads, and logs.
- Unused volumes (not referenced by any container) are removed by `docker system prune --volumes`.

**Example (in `docker-compose.yml`):**
  ```
  volumes:
    - db-data:/var/lib/postgresql/data
  ```

- Managed with commands like:
  ```
  docker volume ls
  docker volume inspect <volume_name>
  ```


### 7. **Bind Mount**
- Maps a directory or file from your host machine directly into the container.
- Changes on the host are reflected in the container (and vice versa).
- Great for development (e.g., mount your source code for live editing).

**Example (in `docker-compose.yml`):**
  ```
  volumes:
    - .:/usr/src/app
  ```

---

## 🌐 Networking Concepts

### 8. **Network**
- Allows Docker containers to communicate with each other.
- By default, Compose creates a network for your app’s services.
- You can define custom networks for more control.
- Docker networks are virtual constructs, but their configuration is stored on disk (e.g., `/var/lib/docker/network/` on Linux).
- Unused custom networks (not used by any container) are removed by `docker system prune`.

---

## 🔄 Lifecycle & Management

### 9. **Build**
- Creates a Docker image from a Dockerfile.
- **Command:**
  ```bash
  docker build -t myimage .
  ```

### 10. **Run**
- Starts a container from an image.
- **Command:**
  ```bash
  docker run myimage
  ```

### 11. **Pull / Push**
- Download or upload images from/to a registry.
- **Commands:**
  ```bash
  docker pull <image>
  docker push <image>
  ```

### 12. **Exec**
- Run a command inside a running container.
- **Example:**
  ```bash
  docker exec -it <container> bash
  ```

---

## 💽 Disk Usage & Cleanup

- **Images** consume the most disk space — especially large base images or many image versions.
  - ⚠️ It's common for a small project to consume multiple gigabytes — e.g., 1.5 GB per image is typical for server or database containers.
    - Example: A simple project with a web server and PostgreSQL can easily take **3 GB or more**, just from shared public images.
    -  Docker is a great way to streamline development with portable, ready-to-go environments — just make sure you have plenty of disk space available!

- **Containers** use a writable layer on disk that grows over time (e.g., logs, temp files).
- **Volumes** store persistent data (e.g., databases, file uploads) and can grow large.
- **Networks** are virtual, but their configurations are stored on disk (e.g., `/var/lib/docker/network/` on Linux).
- **Dangling images**, **stopped containers**, **unused volumes**, and **unused networks** can accumulate over time.

### 🔍 How to check disk usage
```bash
docker system df
```

### 🧹 How to clean up unused resources
```bash
docker system prune
docker system prune --volumes  # Also removes unused volumes
```

> ⚠️ Use with care! Prune deletes *everything* unused — including stopped containers and untagged images.

---

## 🛠️ Useful Docker Commands

| Command                                | Description                                 |
|-----------------------------------------|---------------------------------------------|
| `docker ps`                            | List running containers                    |
| `docker images`                        | List images on your system                  |
| `docker stop <container>`              | Stop a running container                   |
| `docker rm <container>`                | Remove a container                         |
| `docker rmi <image>`                   | Remove an image                            |
| `docker volume ls`                     | List volumes                               |
| `docker-compose up`                    | Start all services in the Compose file     |
| `docker-compose down`                  | Stop and remove services/volumes/networks  |

---

## 📝 Summary Table of Key Concepts

| Concept       | What It Is                             | Example / Use Case              |
|---------------|-----------------------------------------|---------------------------------|
| Image         | Blueprint for containers               | `node:18`, `postgres:16`        |
| Container     | Running instance of an image           | Your app or DB running isolated |
| Dockerfile    | Script to build an image                | Defines app setup              |
| Compose       | Multi-container orchestration          | App + DB together              |
| Registry      | Image storage and distribution         | Docker Hub, AWS ECR            |
| Volume        | Persistent storage (managed by Docker) | Database data, user uploads    |
| Bind Mount    | Host directory/file mapped into container | Live code editing in dev    |
| Network       | Communication between containers       | App talks to DB by name        |

---

## ☕ Buy Me a Coffee

If you found this cheat sheet helpful and want to support my work, you can [buy me a coffee](https://www.buymeacoffee.com/miikapirtola)! 

---

## 📜 License
MIT License.
