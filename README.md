

# Docker Concepts Advanced Cheat Sheet

This cheat sheet summarizes the essential ideas and commands for understanding how Docker works, and helps you understand how Docker uses disk and memory resources in practice.

---

## 🐳 What Is Docker?

- Docker is a technology that lets you package and run applications in a portable, self-contained way — so they behave the same across different machines, operating systems, and environments.
  - For example, you can deploy a web server on a remote VPS you just rented, and it runs exactly as it did on your laptop — no manual dependency setup, no configuration mismatches.
  - You can share a complete development environment with teammates or contributors, so they can get started instantly without installing languages, frameworks, or tools — everything comes bundled inside the container.
- To work with Docker — run containers, build images, manage volumes, etc. — you need to install Docker on your machine.
  - The official installer takes care of everything — it sets up the Docker Engine, command-line tools, and optional desktop app, all in one go.
- Docker is a **Linux-native** application. On macOS and Windows, it runs inside a lightweight Linux virtual machine, which is set up automatically by the Docker installer.
- On all systems, Docker runs as a background service (also called the Docker daemon or Docker Engine).
- You can manage Docker through the command-line interface, or with Docker Desktop, a graphical application that provides a visual way to manage containers, images, and more.

---

## 🧰 Common Ways to Use Docker

### 🧪 Development Environments

#### 🤔 Why Would You Want to Use Docker for Development?

At first glance, it might seem unnecessary to use Docker for simple development projects — especially if you're already managing your dependencies with tools like `npm`, `pip`, or `venv`.

But Docker solves a different class of problems: it gives you a **consistent and isolated runtime environment**, regardless of your host machine.

##### ✅ Key Benefits

- **Same environment everywhere**  
  Docker ensures every developer on your team uses the same Node/Python/etc. version, the same OS, and the same configuration — no more "works on my machine" issues.

- **No need to install anything locally**  
  Your host system stays clean. No global packages, no version conflicts, no need to install specific language runtimes — it's all in the container.

- **Cross-platform compatibility**  
  Whether you're on Windows, macOS, or Linux, the app runs identically inside Docker's Linux-based container.

- **Production-ready by default**  
  The container you develop in can be shipped as-is to production — no surprises at deployment time.

- **CI/CD integration**  
  Most modern CI pipelines use Docker under the hood. Having a `Dockerfile` makes you ready to integrate with testing, building, and deployment workflows.

##### 🧠 Even for a simple app…

You may only be building a web app with Node or Python — but tomorrow you might add:
- A database like PostgreSQL
- A caching layer like Redis
- A background job runner

Docker lets you **grow your project safely** without needing to reconfigure your machine or your dev setup from scratch.

#### 📥 Cloning an Existing Dev Project that Utilizes Docker

- First, make sure Docker is installed and running on your development machine.  
  - On Windows/macOS, Docker Desktop must be running in the background.  
  - On Linux, Docker runs as a system service.

- You can clone a GitHub project that includes Docker support and start working with it right away — without installing anything else manually.

- A typical project will include:
  - A `Dockerfile` that defines how to build the app image — Yes, the filename looks odd like that, with no extension!
  - Optionally, a `docker-compose.yml` file that defines related services like a PostgreSQL database, Redis cache, or other supporting containers.
    - Each service runs in its own container, allowing Docker to run a **composition of containers** that work together as one system.

- After cloning the project, you can start the full stack depending on the setup:
  - If the project includes a `docker-compose.yml` file:
    ```bash
    docker-compose up
    ```
    Or, if the app image needs to be rebuilt:
    ```bash
    docker-compose up --build
    ```
  - If there's only a `Dockerfile`, you can build and run the image manually:
    ```bash
    docker build -t myapp .
    docker run myapp
    ```

- Docker will:
  - Build your app image (if needed).
  - Download any required public images like `postgres`, `redis`, etc.
  - Create and start all defined containers in a shared network.

- Everything runs in containers — no need to install Node, Python, PostgreSQL, etc. on your host system.
  - On **Windows/macOS**, containers and images are stored inside Docker’s internal Linux virtual machine.
  - On **Linux**, they are stored directly on the host, typically under `/var/lib/docker/`.

- Once running, the application and its services are fully isolated and reproducible across machines — every developer gets the same environment.

### 🧱 Adding Docker Support to a Dev Project

You can make an existing development project Docker-ready by adding a few configuration files:

- Add a `Dockerfile` to define how your app is built and run inside a container.  
  This includes setting the base image (like `node`, `python`, or `golang`), copying your source code, and specifying how to start the app.

- Optionally, add a `docker-compose.yml` file to define supporting services like databases or caches.  
  For example, you can configure PostgreSQL, Redis, or other containers to run alongside your app.

Once added, your entire project becomes self-contained and portable — other developers can clone it, run `docker-compose up`, and instantly get the full dev stack running with no manual setup.

### 🚀 Deployment & Hosting

The same image you built for development can be deployed to a production server or cloud provider — no need to reconfigure or rebuild.

- *Example:* Deploy to a VPS and run:
  ```bash
  docker run -d -p 80:80 myapp
  ```

### 🗄️ Running Local Services (like Databases)

Need PostgreSQL, Redis, or Elasticsearch on your machine? No need to install them — just run the official image as a container.

- *Example:* Run a PostgreSQL database container:
  ```bash
  docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres
  ```

### 🔧 CLI Tools & One-Off Scripts

You can run command-line tools like Python, Node, or build tools without installing them on your host system.

- *Example:* Launch a Python shell or run a Node script:
  ```bash
  docker run -it python
  docker run --rm node npx create-react-app
  ```

### 🐧 Linux Shell on Any OS

Docker lets you launch a minimal Linux shell on Windows or macOS instantly — useful for testing, scripting, or Linux-only tools.

- *Example:* Run an Alpine or Ubuntu shell:
  ```bash
  docker run -it alpine
  docker run -it ubuntu bash
  ```
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
