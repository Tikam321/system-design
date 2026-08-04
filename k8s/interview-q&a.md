# Kubernetes Interview Questions & Answers

A comprehensive guide covering Kubernetes concepts from beginner to advanced level, suitable for interview preparation.

---

## Table of Contents
1. [Basic Concepts](#basic-concepts)
2. [Architecture](#architecture)
3. [Pods, Deployments & Workloads](#pods-deployments--workloads)
4. [Networking](#networking)
5. [Storage](#storage)
6. [Configuration & Secrets](#configuration--secrets)
7. [Scheduling & Scaling](#scheduling--scaling)
8. [Security](#security)
9. [Troubleshooting & Operations](#troubleshooting--operations)
10. [Advanced / Scenario-Based Questions](#advanced--scenario-based-questions)

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
