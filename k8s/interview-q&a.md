# Docker & Kubernetes Interview Questions & Answers

A comprehensive guide covering Docker and Kubernetes concepts from beginner to advanced level, suitable for interview preparation.

---

## Table of Contents

**Part 1: Docker**
1. [Docker Basics](#docker-basics)
2. [Docker Images & Containers](#docker-images--containers)
3. [Dockerfile](#dockerfile)
4. [Docker Networking](#docker-networking)
5. [Docker Storage/Volumes](#docker-storagevolumes)
6. [Docker Compose](#docker-compose)
7. [Docker Security & Best Practices](#docker-security--best-practices)
8. [Docker Troubleshooting & Scenario-Based](#docker-troubleshooting--scenario-based)

**Part 2: Kubernetes**
9. [Basic Concepts](#basic-concepts)
10. [Architecture](#architecture)
11. [Pods, Deployments & Workloads](#pods-deployments--workloads)
12. [Networking](#networking)
13. [Storage](#storage)
14. [Configuration & Secrets](#configuration--secrets)
15. [Scheduling & Scaling](#scheduling--scaling)
16. [Security](#security)
17. [Troubleshooting & Operations](#troubleshooting--operations)
18. [Advanced / Scenario-Based Questions](#advanced--scenario-based-questions)

---

# Part 1: Docker

## Docker Basics

### D1. What is Docker?
Docker is a platform for developing, shipping, and running applications inside lightweight, portable containers. Containers package an application with all its dependencies, ensuring it runs consistently across different environments.

### D2. What is a container? How is it different from a virtual machine?
A container is an isolated, lightweight runtime environment that shares the host OS kernel but has its own filesystem, process space, and network stack. Unlike VMs, containers don't need a full guest OS per instance, making them faster to start and more resource-efficient. VMs virtualize hardware (via a hypervisor) and each runs a complete OS, while containers virtualize at the OS level.

### D3. What is the Docker architecture?
Docker uses a client-server architecture:
- **Docker Client** – CLI/API used to issue commands
- **Docker Daemon (dockerd)** – background service that builds, runs, and manages containers
- **Docker Registry** – stores Docker images (e.g., Docker Hub, private registries)
- **containerd/runc** – the underlying container runtime that actually creates and runs containers

### D4. What is the difference between an Image and a Container?
An **image** is a read-only template containing the application code, runtime, libraries, and dependencies. A **container** is a running (or stopped) instance of an image — a writable layer added on top of the image's read-only layers.

### D5. What is Docker Hub?
Docker Hub is a public cloud-based registry service for finding, storing, and sharing container images.

---

## Docker Images & Containers

### D6. How are Docker images structured?
Images are built as a series of read-only, stacked layers, each representing an instruction in the Dockerfile. Layers are cached and shared between images, which saves disk space and speeds up builds.

### D7. What is the Union File System in Docker?
It's the filesystem technique Docker uses to layer multiple directories (image layers) into a single unified view, allowing images to share common layers while a thin writable layer is added on top for the running container.

### D8. What happens to data when a container is deleted?
Any data written to the container's writable layer is lost when the container is removed, unless it was stored in a volume or bind mount, which persist independently of the container lifecycle.

### D9. What is the difference between `CMD` and `ENTRYPOINT`?
- **ENTRYPOINT** – defines the fixed, main command that always runs when the container starts.
- **CMD** – provides default arguments to the ENTRYPOINT, or the default command if no ENTRYPOINT is set; can be overridden at runtime (`docker run image <new-cmd>`).
They're often combined: `ENTRYPOINT` sets the executable, `CMD` sets default flags/args.

### D10. What is the difference between `COPY` and `ADD` in a Dockerfile?
`COPY` simply copies files/directories from the build context into the image. `ADD` does the same but also supports auto-extracting local tar archives and fetching remote URLs. Best practice: prefer `COPY` unless you specifically need `ADD`'s extra behavior.

### D11. What are common Docker commands?
```bash
docker build -t myapp:1.0 .          # build an image
docker run -d -p 8080:80 myapp:1.0   # run a container
docker ps -a                         # list containers
docker images                        # list images
docker exec -it <container> bash     # shell into running container
docker logs -f <container>           # stream logs
docker stop/start/rm <container>     # lifecycle management
docker rmi <image>                   # remove an image
docker inspect <container|image>     # detailed metadata
```

### D12. What is the difference between `docker stop` and `docker kill`?
`docker stop` sends `SIGTERM` to allow graceful shutdown, then `SIGKILL` after a grace period (default 10s). `docker kill` sends `SIGKILL` immediately, terminating the container without cleanup.

### D13. What is the difference between `docker run` and `docker start`?
`docker run` creates a **new** container from an image and starts it. `docker start` restarts an **existing, stopped** container.

---

## Dockerfile

### D14. What is a Dockerfile?
A text file containing a set of instructions used to automatically build a Docker image, specifying the base image, dependencies, files to copy, commands to run, exposed ports, and the startup command.

### D15. What is a multi-stage build and why use it?
Multi-stage builds use multiple `FROM` statements in a single Dockerfile, where each stage can use a different base image. Artifacts are selectively copied from one stage to another (`COPY --from=<stage>`), allowing you to compile/build in a heavy image but ship only the compiled artifact in a minimal final image — significantly reducing final image size.

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM alpine:latest
COPY --from=builder /app/myapp /usr/local/bin/myapp
ENTRYPOINT ["myapp"]
```

### D16. How do you reduce Docker image size?
- Use minimal base images (`alpine`, `distroless`)
- Use multi-stage builds
- Combine `RUN` commands to reduce layers and clean up in the same layer (e.g., `apt-get update && apt-get install -y x && rm -rf /var/lib/apt/lists/*`)
- Use `.dockerignore` to exclude unnecessary files from the build context
- Avoid installing unnecessary packages/dev dependencies in the final image

### D17. What is the purpose of `.dockerignore`?
Similar to `.gitignore`, it specifies files/directories to exclude from the Docker build context, reducing build time and image size, and preventing sensitive files from being accidentally included.

### D18. What is Docker layer caching and how does it affect build order?
Docker caches each image layer; if a layer's instruction and context haven't changed, Docker reuses the cached layer instead of rebuilding it. Best practice: place instructions that change less frequently (like dependency installation) before those that change often (like copying source code), so cache is invalidated as late as possible.

### D19. What does `EXPOSE` do in a Dockerfile?
It documents which port(s) the container listens on at runtime. It does **not** actually publish the port — that requires `-p`/`--publish` at `docker run` time.

---

## Docker Networking

### D20. What are the default Docker network types?
- **bridge** (default) – private internal network on the host; containers communicate via this virtual bridge
- **host** – container shares the host's network namespace directly (no isolation)
- **none** – container has no network access
- **overlay** – enables communication between containers across multiple Docker hosts (used in Swarm)
- **macvlan** – assigns a MAC address to a container, making it appear as a physical device on the network

### D21. How do containers communicate with each other?
On the same user-defined bridge network, containers can reach each other using their container name as a DNS hostname. Docker's embedded DNS server resolves these names automatically.

### D22. How do you expose a container's port to the host?
```bash
docker run -p <host_port>:<container_port> myimage
```

---

## Docker Storage/Volumes

### D23. What are the ways to persist data in Docker?
- **Volumes** – managed by Docker, stored in `/var/lib/docker/volumes/`, the recommended approach
- **Bind mounts** – map a specific host filesystem path directly into the container
- **tmpfs mounts** – stored in host memory only, never written to disk (useful for sensitive/temporary data)

### D24. What is the difference between a Volume and a Bind Mount?
Volumes are fully managed by Docker and are portable/decoupled from host directory structure. Bind mounts depend on the host machine's specific filesystem layout, offering more direct control but less portability.

---

## Docker Compose

### D25. What is Docker Compose?
A tool for defining and running multi-container Docker applications using a single YAML file (`docker-compose.yml`), allowing you to configure services, networks, and volumes, and spin everything up with one command (`docker compose up`).

### D26. Example docker-compose.yml
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

### D27. What does `depends_on` do in Docker Compose?
It controls startup **order** of services (ensuring one container starts before another), but by default does not wait for the dependency to be fully "ready" — only for it to have started. For readiness, use healthchecks combined with `depends_on: condition: service_healthy`.

---

## Docker Security & Best Practices

### D28. How do you run a container as a non-root user?
```dockerfile
RUN useradd -m appuser
USER appuser
```
This limits the blast radius if the container is compromised, since processes don't run with root privileges.

### D29. What is Docker Content Trust?
A security feature that enables digital signing and verification of images, ensuring only signed, trusted images are pulled and run (`DOCKER_CONTENT_TRUST=1`).

### D30. What are some Docker security best practices?
- Use minimal/official base images and scan them for vulnerabilities (Trivy, Docker Scout, Snyk)
- Avoid running containers as root
- Don't store secrets in images/Dockerfiles — use secret managers or runtime injection
- Keep images updated with security patches
- Use read-only filesystems where possible (`--read-only`)
- Limit container capabilities (`--cap-drop=ALL`, add back only what's needed)
- Set resource limits (`--memory`, `--cpus`) to prevent resource exhaustion attacks

### D31. What is the difference between `docker save` and `docker export`?
- `docker save` – exports an **image** (with all layers and metadata/history) to a tarball, can be reloaded with `docker load`.
- `docker export` – exports a **container's** filesystem (flattened, no layer history) to a tarball, restored with `docker import`.

---

## Docker Troubleshooting & Scenario-Based

### D32. A container exits immediately after starting — how do you debug it?
```bash
docker logs <container>
docker inspect <container>   # check ExitCode, check for OOMKilled
```
Common causes: the main process completes/crashes immediately, misconfigured `CMD`/`ENTRYPOINT`, missing environment variables, or the process expects a foreground process but exits when a background daemon finishes.

### D33. How do you check resource usage of running containers?
```bash
docker stats
```
Shows live CPU, memory, network I/O, and block I/O usage per container.

### D34. How do you clean up unused Docker resources?
```bash
docker system prune -a         # remove unused containers, networks, images
docker volume prune            # remove unused volumes
docker container prune         # remove stopped containers
```

### D35. What's the difference between Docker Swarm and Kubernetes?
Both are orchestration platforms, but Docker Swarm is simpler to set up and tightly integrated with Docker CLI, suited for smaller-scale deployments. Kubernetes offers a richer feature set (auto-scaling, self-healing, advanced scheduling, huge ecosystem) and is the industry standard for large-scale, production-grade container orchestration, at the cost of a steeper learning curve.

### D36. How would you troubleshoot high memory usage that leads to a container being OOMKilled?
Check `docker inspect` for `OOMKilled: true`, review the app's actual memory consumption via `docker stats`/APM tools, verify if a memory limit (`--memory`) is set too low, check for memory leaks in the application, and consider increasing limits or optimizing the application.

---

# Part 2: Kubernetes

---

## Basic Concepts

### 1. What is Kubernetes?
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. Originally developed by Google and now maintained by the CNCF, it groups containers into logical units for easy management and discovery.

### 2. What problems does Kubernetes solve?
- Manual scaling and deployment of containers across many hosts
- Service discovery and load balancing
- Self-healing (restarting failed containers, rescheduling on node failure)
- Automated rollouts and rollbacks
- Secret and configuration management
- Efficient resource utilization via bin-packing

### 3. What is the difference between Docker and Kubernetes?
Docker is a containerization platform used to build and run containers on a single host. Kubernetes is an orchestration system that manages containers (which can be Docker, containerd, CRI-O, etc.) across a cluster of multiple hosts, handling scaling, networking, and failover.

### 4. What is a Cluster in Kubernetes?
A cluster is a set of nodes (machines) that run containerized applications managed by Kubernetes. It consists of a **control plane** (manages the cluster state) and **worker nodes** (run the actual workloads).

### 5. What is a Namespace?
A namespace is a virtual cluster within a physical cluster, used to divide cluster resources between multiple users, teams, or projects. Common default namespaces include `default`, `kube-system`, and `kube-public`.

### 6. What is kubectl?
`kubectl` is the command-line tool used to interact with the Kubernetes API server — to deploy applications, inspect and manage cluster resources, and view logs.

---

## Architecture

### 7. Explain the Kubernetes architecture.
Kubernetes follows a master-worker (control plane–node) architecture:

**Control Plane components:**
- **kube-apiserver** – front-end for the Kubernetes control plane; exposes the REST API
- **etcd** – consistent, distributed key-value store holding all cluster state
- **kube-scheduler** – assigns newly created pods to nodes based on resource requirements and constraints
- **kube-controller-manager** – runs controller processes (node controller, replication controller, endpoints controller, etc.)
- **cloud-controller-manager** – integrates with underlying cloud provider APIs

**Node (Worker) components:**
- **kubelet** – agent that ensures containers are running in a pod as expected
- **kube-proxy** – maintains network rules for pod communication and load balancing
- **Container runtime** – software that runs containers (containerd, CRI-O, etc.)

### 8. What is etcd and why is it important?
etcd is a distributed, consistent key-value store used as Kubernetes' backing store for all cluster data — configuration, state, and metadata. If etcd is lost or corrupted without a backup, the entire cluster state is lost, so regular backups are critical.

### 9. What does the kube-scheduler do?
It watches for newly created pods with no assigned node and selects a node for them to run on, based on factors like resource requirements, hardware/software constraints, affinity/anti-affinity rules, taints/tolerations, and data locality.

### 10. What is the role of the kubelet?
The kubelet runs on every worker node and ensures that containers described in PodSpecs are running and healthy. It communicates with the container runtime and reports node/pod status back to the API server.

### 11. What is kube-proxy?
kube-proxy runs on each node and maintains network rules that allow network communication to pods from inside or outside the cluster. It implements part of the Kubernetes Service concept, typically using iptables or IPVS.

---

## Pods, Deployments & Workloads

### 12. What is a Pod?
A Pod is the smallest deployable unit in Kubernetes. It represents one or more containers that share the same network namespace (IP address) and storage volumes, and are always scheduled together on the same node.

### 13. Why would a Pod have multiple containers?
Multi-container pods are used for tightly coupled helper processes, such as:
- **Sidecar** – e.g., a logging agent that ships logs from the main container
- **Ambassador** – proxies network requests to external services
- **Adapter** – standardizes/transforms output from the main container

### 14. What is a ReplicaSet?
A ReplicaSet ensures a specified number of pod replicas are running at any given time. It's usually not managed directly, but through a Deployment.

### 15. What is a Deployment?
A Deployment provides declarative updates for Pods and ReplicaSets. It manages rolling updates, rollbacks, and scaling, and ensures the desired state matches the actual state of the cluster.

### 16. What is the difference between a Deployment, StatefulSet, and DaemonSet?
| Resource | Use Case | Key Characteristic |
|---|---|---|
| **Deployment** | Stateless apps | Pods are interchangeable, no stable identity |
| **StatefulSet** | Stateful apps (databases, queues) | Stable network identity, stable storage, ordered deployment/scaling |
| **DaemonSet** | Node-level agents (log collectors, monitoring agents) | Runs exactly one pod per (matching) node |

### 17. What is a Job and a CronJob?
- **Job** – creates one or more pods and ensures a specified number of them successfully terminate (used for batch/one-off tasks).
- **CronJob** – creates Jobs on a repeating schedule, similar to a Unix cron.

### 18. What is the Pod lifecycle?
A Pod goes through these phases: `Pending` → `Running` → `Succeeded`/`Failed`. Within `Running`, containers can be in `Waiting`, `Running`, or `Terminated` states. Kubelet also runs **readiness**, **liveness**, and **startup** probes to determine container health.

### 19. What are Liveness, Readiness, and Startup probes?
- **Liveness probe** – checks if a container is still running correctly; if it fails, the container is restarted.
- **Readiness probe** – checks if a container is ready to accept traffic; if it fails, the pod is removed from Service endpoints.
- **Startup probe** – used for slow-starting containers, disables liveness/readiness checks until the app has started.

### 20. What happens when a Pod is deleted?
Kubernetes sends a `SIGTERM` to the container, waits for the `terminationGracePeriodSeconds` (default 30s) to allow graceful shutdown, then sends `SIGKILL` if the container hasn't exited.

### 21. How do rolling updates work in a Deployment?
Kubernetes incrementally replaces old pods with new ones based on the `RollingUpdateStrategy`, controlled by `maxUnavailable` and `maxSurge` parameters, ensuring zero (or minimal) downtime.

### 22. How do you roll back a Deployment?
```bash
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=<n>
kubectl rollout history deployment/<name>
```

---

## Networking

### 23. Explain the Kubernetes networking model.
Kubernetes requires:
1. Every pod gets its own unique IP address.
2. Pods can communicate with all other pods without NAT.
3. Nodes can communicate with all pods without NAT.
This is typically implemented via a CNI (Container Network Interface) plugin like Calico, Flannel, Cilium, or Weave.

### 24. What is a Service in Kubernetes?
A Service is an abstraction that defines a logical set of Pods (usually via label selectors) and a policy to access them — providing a stable IP/DNS name even as underlying pods are created/destroyed.

### 25. What are the types of Kubernetes Services?
- **ClusterIP** (default) – exposes the service internally within the cluster
- **NodePort** – exposes the service on a static port on each node's IP
- **LoadBalancer** – provisions an external load balancer (cloud provider dependent)
- **ExternalName** – maps a service to an external DNS name, no proxying

### 26. What is an Ingress?
Ingress manages external HTTP/HTTPS access to services in a cluster, providing load balancing, SSL termination, and name-based virtual hosting. It requires an Ingress Controller (e.g., NGINX, Traefik, ALB) to function.

### 27. What is the difference between a Service and an Ingress?
A Service exposes a set of pods (typically L4 — TCP/UDP), while Ingress operates at L7 (HTTP/HTTPS) and can route based on hostnames/paths, consolidating multiple services behind a single external IP.

### 28. How does DNS work inside a Kubernetes cluster?
CoreDNS (or kube-dns) runs as a cluster add-on and provides DNS resolution for Services and Pods, typically in the format `<service-name>.<namespace>.svc.cluster.local`.

### 29. What is a Network Policy?
A NetworkPolicy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints. By default, all pods can communicate freely; NetworkPolicies restrict this (requires a CNI plugin that supports them).

---

## Storage

### 30. What is a Volume in Kubernetes?
A Volume is a directory accessible to containers in a pod, with a lifecycle tied to the pod (unlike container-local storage, which is ephemeral). Types include `emptyDir`, `hostPath`, `configMap`, `secret`, and cloud-based/persistent volumes.

### 31. What is the difference between PersistentVolume (PV) and PersistentVolumeClaim (PVC)?
- **PV** – a piece of storage in the cluster provisioned by an admin or dynamically via a StorageClass.
- **PVC** – a request for storage by a user, which binds to a matching PV.

### 32. What is a StorageClass?
A StorageClass defines a "class" of storage (e.g., SSD vs HDD, provisioner type) and enables **dynamic provisioning** of PersistentVolumes on demand, instead of requiring pre-created PVs.

### 33. What access modes exist for PVs?
- **ReadWriteOnce (RWO)** – read-write by a single node
- **ReadOnlyMany (ROX)** – read-only by many nodes
- **ReadWriteMany (RWX)** – read-write by many nodes
- **ReadWriteOncePod (RWOP)** – read-write by a single pod only

---

## Configuration & Secrets

### 34. What is a ConfigMap?
A ConfigMap stores non-confidential configuration data as key-value pairs, which can be consumed by pods as environment variables, command-line arguments, or mounted files.

### 35. What is a Secret? How is it different from a ConfigMap?
A Secret stores sensitive data (passwords, tokens, keys), base64-encoded (not encrypted by default at rest unless encryption-at-rest is configured). Functionally similar to ConfigMaps in usage, but intended for sensitive data with tighter access controls (RBAC).

### 36. How can you make Secrets more secure?
- Enable encryption at rest for etcd
- Use RBAC to restrict access
- Use external secret managers (HashiCorp Vault, AWS Secrets Manager, Sealed Secrets, External Secrets Operator)
- Avoid checking secrets into version control

---

## Scheduling & Scaling

### 37. How does the scheduler decide where to place a pod?
Through a two-step process:
1. **Filtering** – eliminates nodes that don't meet requirements (resources, taints, affinity rules)
2. **Scoring** – ranks remaining nodes and picks the best fit

### 38. What are Taints and Tolerations?
- **Taint** – applied to a node to repel pods that don't tolerate it (`kubectl taint nodes node1 key=value:NoSchedule`)
- **Toleration** – applied to a pod to allow (but not require) scheduling onto nodes with matching taints

### 39. What is Node Affinity vs Pod Affinity/Anti-Affinity?
- **Node Affinity** – constrains which nodes a pod can be scheduled on, based on node labels.
- **Pod Affinity** – schedules pods close to other pods (e.g., same node/zone).
- **Pod Anti-Affinity** – ensures pods are spread apart (e.g., not on the same node), useful for high availability.

### 40. What is Horizontal Pod Autoscaler (HPA)?
HPA automatically scales the number of pod replicas based on observed metrics like CPU/memory utilization or custom metrics.

### 41. What is Vertical Pod Autoscaler (VPA)?
VPA automatically adjusts the CPU/memory **requests and limits** of containers based on usage history, rather than changing the number of replicas.

### 42. What is Cluster Autoscaler?
It automatically adjusts the number of **nodes** in a cluster — adding nodes when pods fail to schedule due to resource shortage, and removing underutilized nodes.

### 43. What are Resource Requests and Limits?
- **Requests** – minimum resources guaranteed to a container; used by the scheduler for placement.
- **Limits** – maximum resources a container can use; enforced by the kubelet/runtime (can lead to throttling or OOMKill).

### 44. What are Quality of Service (QoS) classes for pods?
- **Guaranteed** – requests = limits for all containers
- **Burstable** – at least one container has requests < limits
- **BestEffort** – no requests/limits set at all
QoS class determines eviction priority under resource pressure (BestEffort evicted first).

---

## Security

### 45. What is RBAC in Kubernetes?
Role-Based Access Control governs who can perform what actions on which resources. Key objects: `Role`/`ClusterRole` (define permissions) and `RoleBinding`/`ClusterRoleBinding` (assign permissions to users/groups/service accounts).

### 46. What is a ServiceAccount?
A ServiceAccount provides an identity for processes running in a pod to authenticate with the Kubernetes API server, distinct from a regular user account.

### 47. What is a Pod Security Standard/Admission?
Pod Security Admission enforces security policies on pods at the namespace level (replacing the deprecated PodSecurityPolicy), with three levels: `Privileged`, `Baseline`, and `Restricted`.

### 48. How do you secure the Kubernetes API server?
- Enable authentication (certificates, tokens, OIDC)
- Use RBAC for authorization
- Enable audit logging
- Use admission controllers (e.g., `PodSecurity`, `NetworkPolicy`)
- Restrict anonymous access and expose the API server only over TLS

### 49. What are Admission Controllers?
Plugins that intercept requests to the API server after authentication/authorization but before object persistence — they can validate or mutate the request (e.g., `MutatingAdmissionWebhook`, `ValidatingAdmissionWebhook`, `ResourceQuota`).

---

## Troubleshooting & Operations

### 50. A pod is stuck in `Pending` state — how do you debug it?
```bash
kubectl describe pod <pod-name>
```
Common causes: insufficient cluster resources, unsatisfied node affinity/taints, no matching PV for a PVC, or image pull issues at a later stage.

### 51. A pod is in `CrashLoopBackOff` — how do you troubleshoot?
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```
Common causes: application crashing on startup, failing liveness probe, misconfiguration, missing dependencies/environment variables.

### 52. How do you check logs of a pod with multiple containers?
```bash
kubectl logs <pod-name> -c <container-name>
```

### 53. How do you debug a service that isn't reachable?
1. Check the Service has correct selectors matching pod labels: `kubectl get endpoints <svc>`
2. Verify pods are `Running` and `Ready`
3. Check NetworkPolicies aren't blocking traffic
4. Test with `kubectl exec` into a pod using `curl`/`wget` to the service's ClusterIP or DNS name

### 54. How do you perform a graceful node maintenance/drain?
```bash
kubectl cordon <node>     # mark unschedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
# ... perform maintenance ...
kubectl uncordon <node>
```

### 55. How do you back up and restore etcd?
```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=<endpoint> --cacert=<ca> --cert=<cert> --key=<key>

ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```

### 56. What tools are commonly used to monitor Kubernetes clusters?
Prometheus + Grafana (metrics), Fluentd/Fluent Bit + Elasticsearch/Loki (logging), Jaeger/Zipkin (tracing), kube-state-metrics, and metrics-server (for HPA).

---

## Advanced / Scenario-Based Questions

### 57. How would you design a zero-downtime deployment strategy?
- Use `RollingUpdate` strategy with sensible `maxUnavailable`/`maxSurge`
- Configure readiness probes correctly so traffic isn't sent to unready pods
- Use `PodDisruptionBudgets` to limit voluntary disruptions during maintenance
- Consider Blue-Green or Canary deployments (via tools like Argo Rollouts or Flagger) for safer rollouts

### 58. What is a PodDisruptionBudget (PDB)?
A PDB limits the number of pods of a replicated application that can be down simultaneously during voluntary disruptions (like node drains), helping maintain application availability.

### 59. How does Kubernetes handle a node failure?
The node controller detects the node is unreachable (missed heartbeats), marks it `NotReady`, and after a grace period, pods are evicted and rescheduled onto healthy nodes (assuming they're managed by a controller like a Deployment or ReplicaSet — standalone pods are not rescheduled).

### 60. What is the difference between `kubectl apply` and `kubectl create`?
`kubectl create` imperatively creates a resource and fails if it already exists. `kubectl apply` is declarative — it creates the resource if it doesn't exist, or patches it to match the provided manifest if it does, and tracks changes via annotations for three-way merges.

### 61. Explain Custom Resource Definitions (CRDs) and Operators.
- **CRD** – extends the Kubernetes API to define custom resource types beyond the built-ins (e.g., `Database`, `Certificate`).
- **Operator** – a controller that watches CRDs and encodes operational knowledge (e.g., backup, upgrade, failover logic) to manage complex applications automatically, extending Kubernetes' native reconciliation loop pattern.

### 62. What is the reconciliation loop / controller pattern?
Controllers continuously watch the actual state of the cluster (via the API server) and compare it to the desired state defined in the spec, taking corrective action to converge actual state toward desired state. This is the core operating principle behind Deployments, ReplicaSets, and Operators.

### 63. How would you handle multi-tenancy in a single Kubernetes cluster?
- Use Namespaces per tenant with ResourceQuotas and LimitRanges
- Apply NetworkPolicies to isolate traffic between tenants
- Use RBAC to scope tenant permissions
- Consider dedicated node pools with taints/tolerations for stronger isolation
- For stricter isolation, consider separate clusters or virtual clusters (vcluster)

### 64. What is the difference between a StatefulSet and a Deployment when a pod is deleted/rescheduled?
In a Deployment, a new pod gets a new name and (typically) new storage. In a StatefulSet, the replacement pod retains the same ordinal name (e.g., `web-0`) and reattaches to the same PersistentVolumeClaim, preserving identity and data.

### 65. How does Kubernetes achieve high availability for the control plane?
Run multiple replicas of the API server behind a load balancer, run etcd as a clustered odd-numbered quorum (e.g., 3 or 5 nodes) across failure domains, and run multiple instances of the scheduler and controller-manager (using leader election, only one is active at a time).

### 66. What is the difference between `emptyDir` and `hostPath` volumes?
- **emptyDir** – created when a pod is assigned to a node, exists as long as the pod runs on that node; data is lost when the pod is removed.
- **hostPath** – mounts a file/directory from the host node's filesystem directly into the pod; data persists on that node but isn't portable across nodes and carries security risks.

### 67. How do you perform canary deployments in Kubernetes?
Run two Deployments (stable and canary) behind the same Service, controlling the proportion of traffic by adjusting the ratio of pod replicas, or use a service mesh (Istio/Linkerd) or tools like Argo Rollouts/Flagger for fine-grained, metric-driven traffic shifting.

### 68. What happens during `kubectl apply -f` under the hood?
The manifest is sent to the API server, validated by admission controllers, persisted to etcd, and then relevant controllers (e.g., Deployment controller) observe the change and drive the cluster toward the new desired state through their reconciliation loops.

---

## Quick Reference: Common kubectl Commands

```bash
kubectl get pods -A                          # list pods in all namespaces
kubectl describe pod <name>                  # detailed pod info/events
kubectl logs -f <pod> -c <container>         # stream container logs
kubectl exec -it <pod> -- /bin/sh            # shell into a pod
kubectl apply -f manifest.yaml               # declarative apply
kubectl scale deployment <name> --replicas=5 # manual scaling
kubectl rollout status deployment/<name>     # check rollout progress
kubectl top pods                             # resource usage (needs metrics-server)
kubectl get events --sort-by='.lastTimestamp' # recent cluster events
```

---

*Good luck with your interview! Tip: Be ready to draw the architecture diagram from memory and explain the request flow from `kubectl apply` all the way to a running pod — this is one of the most commonly asked whiteboard questions.*
