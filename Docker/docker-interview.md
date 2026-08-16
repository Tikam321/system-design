# Docker — Commonly Used Commands & Interview Guide

A practical reference for day-to-day Docker usage, organized by task, followed by common interview questions with concise answers.

---

## 1. Image Commands

| Command | Use |
|---|---|
| `docker build -t myapp:1.0 .` | Build an image from a Dockerfile in the current directory, tagged `myapp:1.0` |
| `docker build -f Dockerfile.dev -t myapp:dev .` | Build using a specific Dockerfile |
| `docker images` | List all local images |
| `docker rmi myapp:1.0` | Remove an image |
| `docker rmi $(docker images -q)` | Remove all local images |
| `docker tag myapp:1.0 myrepo/myapp:1.0` | Tag an image for pushing to a registry |
| `docker push myrepo/myapp:1.0` | Push an image to a registry (Docker Hub, ECR, etc.) |
| `docker pull nginx:latest` | Download an image from a registry |
| `docker history myapp:1.0` | Show the layers that make up an image |
| `docker save -o myapp.tar myapp:1.0` | Export an image to a tar file |
| `docker load -i myapp.tar` | Import an image from a tar file |

---

## 2. Container Lifecycle Commands

| Command | Use |
|---|---|
| `docker run -d -p 3000:3000 --name myapp myapp:1.0` | Run a container in the background (`-d`), map port 3000, give it a name |
| `docker run -it ubuntu bash` | Run interactively with a terminal attached — common for debugging |
| `docker run --rm myapp:1.0` | Run and auto-remove the container when it exits (good for one-off tasks) |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers, including stopped ones |
| `docker stop myapp` | Gracefully stop a running container |
| `docker start myapp` | Start a stopped container |
| `docker restart myapp` | Restart a container |
| `docker kill myapp` | Force-stop a container immediately |
| `docker rm myapp` | Remove a stopped container |
| `docker rm -f myapp` | Force-remove a running container |

---

## 3. Inspecting & Debugging

| Command | Use |
|---|---|
| `docker logs myapp` | View a container's logs |
| `docker logs -f myapp` | Follow logs in real time (like `tail -f`) |
| `docker exec -it myapp bash` | Open a shell inside a running container |
| `docker inspect myapp` | Full JSON details (network, mounts, env vars, etc.) |
| `docker top myapp` | Show running processes inside a container |
| `docker stats` | Live CPU/memory/network usage for running containers |
| `docker port myapp` | Show port mappings for a container |
| `docker diff myapp` | Show filesystem changes made inside a container |

---

## 4. Volumes & Data

| Command | Use |
|---|---|
| `docker volume create mydata` | Create a named volume for persistent storage |
| `docker volume ls` | List volumes |
| `docker volume rm mydata` | Remove a volume |
| `docker run -v mydata:/var/lib/mysql mysql` | Mount a named volume into a container |
| `docker run -v $(pwd):/app myapp` | Bind-mount a local directory into a container (common in local dev) |
| `docker volume prune` | Remove all unused volumes |

---

## 5. Networking

| Command | Use |
|---|---|
| `docker network ls` | List networks |
| `docker network create mynet` | Create a custom network so containers can talk by name |
| `docker run --network mynet myapp` | Attach a container to a specific network |
| `docker network inspect mynet` | See which containers are on a network |
| `docker network rm mynet` | Remove a network |

---

## 6. Cleanup Commands

| Command | Use |
|---|---|
| `docker system df` | Show disk usage by images/containers/volumes |
| `docker system prune` | Remove unused containers, networks, and dangling images |
| `docker system prune -a` | Also remove all unused images (more aggressive) |
| `docker container prune` | Remove all stopped containers |
| `docker image prune` | Remove dangling (untagged) images |

---

## 7. Docker Compose (multi-container apps)

| Command | Use |
|---|---|
| `docker compose up` | Start all services defined in `docker-compose.yml` |
| `docker compose up -d` | Start in detached mode |
| `docker compose up --build` | Rebuild images before starting |
| `docker compose down` | Stop and remove containers, networks (add `-v` to also remove volumes) |
| `docker compose ps` | List services and their status |
| `docker compose logs -f` | Follow logs for all services |
| `docker compose exec web bash` | Shell into a running service named `web` |
| `docker compose restart web` | Restart a single service |

---

## 8. Common Real-World Patterns

**Build, run, and check logs during development:**
```bash
docker build -t myapp .
docker run -d -p 8080:8080 --name myapp myapp
docker logs -f myapp
```

**Debug a container that keeps crashing:**
```bash
docker logs myapp
docker run -it --entrypoint bash myapp   # override CMD to inspect manually
```

**Clean up disk space (common when Docker eats disk):**
```bash
docker system df
docker system prune -a --volumes
```

**Push an image to a private registry (e.g., AWS ECR):**
```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
docker tag myapp:1.0 <account>.dkr.ecr.<region>.amazonaws.com/myapp:1.0
docker push <account>.dkr.ecr.<region>.amazonaws.com/myapp:1.0
```

---

## 9. Interview Questions

### Fundamentals

**Q: What is the difference between an image and a container?**
An image is a read-only, immutable template (built from a Dockerfile) containing the app, dependencies, and config. A container is a running (or stopped) instance of an image — the writable, live version.

**Q: What is a Dockerfile layer, and why does layer order matter?**
Each instruction in a Dockerfile (`RUN`, `COPY`, etc.) creates a cached layer. Docker reuses unchanged layers on rebuild, so placing rarely-changing steps (like dependency installation) before frequently-changing ones (like copying source code) speeds up builds significantly.

**Q: What's the difference between `CMD` and `ENTRYPOINT`?**
`CMD` sets a default command that can be fully overridden by arguments passed to `docker run`. `ENTRYPOINT` sets a fixed command that `docker run` arguments only append to. They're often combined: `ENTRYPOINT` for the fixed executable, `CMD` for default arguments.

**Q: What's the difference between `COPY` and `ADD`?**
`COPY` simply copies files/directories. `ADD` does that plus can extract local tar archives and fetch remote URLs. Best practice is to use `COPY` unless you specifically need `ADD`'s extra behavior.

### Images & Builds

**Q: How do you reduce Docker image size?**
- Use a minimal base image (`alpine`, `slim` variants)
- Use multi-stage builds to discard build tools from the final image
- Combine `RUN` commands to reduce layers
- Add a `.dockerignore` to avoid copying unnecessary files
- Clean up package manager caches in the same `RUN` step

**Q: What is a multi-stage build and why use it?**
It uses multiple `FROM` instructions in one Dockerfile — one stage compiles/builds the app (with all build tools), and a final lean stage copies only the compiled artifacts. This keeps the production image small and free of build-time dependencies.

**Q: What's the difference between `docker build` caching working and not working?**
Docker caches each layer based on the instruction and its context (e.g., file checksums for `COPY`). If any file referenced by an instruction changes, that layer and every layer after it are rebuilt — which is why dependency installs should be ordered before source code copies.

### Containers & Runtime

**Q: What happens when you run `docker run` vs `docker start`?**
`docker run` creates a *new* container from an image and starts it. `docker start` restarts an *existing, stopped* container using its previous configuration.

**Q: How do you persist data across container restarts?**
Use volumes (`docker volume create` + `-v`) or bind mounts. Without them, any data written inside a container's writable layer is lost when the container is removed.

**Q: What's the difference between a volume and a bind mount?**
A volume is managed by Docker and stored in Docker's own storage area — portable and decoupled from host filesystem structure. A bind mount maps a specific host path directly into the container — useful for local development (live code reload) but tightly coupled to the host.

**Q: How does container networking work by default?**
By default, containers on the same user-defined bridge network can reach each other by container name (Docker's internal DNS). Containers on the default `bridge` network can't resolve each other by name unless explicitly linked or placed on a custom network.

### Production & Operations

**Q: How do you run a container as a non-root user, and why does it matter?**
Add a `USER` instruction in the Dockerfile (or `--user` flag at runtime). Running as non-root limits the blast radius if the container is compromised, since the process inside won't have root privileges even if it escapes container isolation.

**Q: What's the purpose of `HEALTHCHECK`?**
It lets Docker (and orchestrators like ECS/Kubernetes) determine if a container is actually functioning correctly, not just "running." A container can be in the `Up` state while its app inside is deadlocked — a health check catches that.

**Q: How do you handle secrets in Docker (e.g., API keys, DB passwords)?**
Never bake secrets into the image via `ENV` or `ARG` — they persist in image layers and are visible via `docker history`. Instead, inject secrets at runtime via environment variables passed to `docker run`/Compose, mounted secret files, or a secrets manager (AWS Secrets Manager, Vault, Docker Swarm/Kubernetes secrets).

**Q: What's the difference between `docker-compose` and Kubernetes?**
Compose is for defining and running multi-container apps on a single host — great for local dev and small deployments. Kubernetes is a full container orchestration platform for running and scaling containers across a cluster of machines, with built-in self-healing, scaling, service discovery, and rolling updates.

**Q: Why might a container exit immediately after starting?**
Common causes: the main process (`CMD`/`ENTRYPOINT`) finished or crashed (Docker containers only stay alive as long as PID 1 runs), a config/env variable is missing causing the app to fail on startup, or the command was overridden incorrectly. `docker logs <container>` is the first place to check.

**Q: How do you debug a container that won't start at all?**
```bash
docker run -it --entrypoint bash myimage   # override default command to get a shell
docker logs <container_id>                 # check exit logs
docker inspect <container_id>              # check exit code, mounts, env
```

**Q: What is the difference between `docker stop` and `docker kill`?**
`docker stop` sends `SIGTERM` (graceful shutdown, then `SIGKILL` after a timeout, default 10s). `docker kill` sends `SIGKILL` immediately, with no chance for cleanup — used when a container is unresponsive.
