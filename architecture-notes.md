# Architecture Notes: Virtual Machines vs Containers

## Comparison Table

| Topic | Virtual Machines (VMs) | Containers |
|-------|--------------------------|------------|
| Kernel Sharing | Each VM runs its own full OS and kernel | Share the host OS kernel |
| Startup Time | Slow (minutes) – full OS boot | Fast (seconds or less) |
| Resource Overhead | High – includes full guest OS | Low – lightweight processes |
| Security Isolation | Strong isolation (hardware-level virtualization) | Process-level isolation (weaker than VM) |
| Operational Complexity | More complex to manage and patch OS per VM | Simpler, especially with orchestration (Kubernetes) |
| Portability | Portable but larger images | Highly portable and lightweight |
| Performance | Slightly higher overhead | Near-native performance |

---

## When would you prefer a VM over a container?

You would prefer a VM when:

- You need strong security isolation (multi-tenant environments).
- You must run different operating systems on the same host.
- You require strict compliance or regulatory isolation.
- You are running legacy applications that depend on a full OS.
- Kernel-level customization is required.

---

## When would you combine both?

You combine VMs and containers when:

- Running Kubernetes clusters on cloud infrastructure (K8s nodes are VMs).
- You want hardware-level isolation between environments, but container flexibility inside.
- You need secure workload separation at infrastructure level while maintaining fast application deployment.
- You operate in enterprise environments where infrastructure teams manage VMs and application teams deploy containers.

A common modern architecture is:

Cloud Infrastructure → Virtual Machines → Kubernetes → Containers

This provides both strong isolation and operational efficiency.


## Step 5

When a pod is deleted or crashes, the Deployment recreates it because it ensures that the number of running pods matches the number of replicas defined in the YAML file.

If no nodes are available with enough resources, the pods remain in the "Pending" state. Kubernetes cannot schedule them until a healthy node with sufficient CPU and memory becomes available. If all nodes are down and no resources exist, the pods cannot run and the application becomes unavailable. In cloud environments with autoscaling, new nodes can be created automatically to accommodate pending pods.



## Kubernetes and Virtualization

### 1️⃣ What runs underneath your k3s cluster?

- k3s runs on **nodes**, which are usually **virtual machines** or physical machines.  
- Each node provides CPU, memory, storage, and networking for the pods.  
- In local setups (minikube, k3s on laptop), the node itself is often a **VM** managed by software like VirtualBox, QEMU, or Docker Desktop.

---

### 2️⃣ Is Kubernetes replacing virtualization?

- **No.** Kubernetes does **not replace virtualization**.  
- Kubernetes is a **container orchestrator**.  
- It schedules and manages containers on nodes, but the nodes themselves still run on **VMs or physical machines**.  
- Containers share the OS kernel, whereas VMs include a full guest OS.

---

### 3️⃣ In a cloud provider, what actually hosts your nodes?

- In AWS, GCP, Azure, or similar:  
  - Nodes are **virtual machines** (EC2 instances, Compute Engine VMs, Azure VMs).  
  - Kubernetes schedules pods onto these VMs.  
  - The cloud provider manages the underlying hardware, networking, and storage.

---

### 4️⃣ Example stack in different environments

#### A) Cloud Data Center


## Production Architecture Design

This section describes a production-ready architecture for a Kubernetes-based quote application system.

---

### 1️⃣ Multiple Nodes

- **Cluster:** At least 3 worker nodes + control plane nodes for HA (High Availability).  
- **Purpose:** Distribute pods across nodes for fault tolerance and load balancing.  
- **Kubernetes role:** Scheduler places pods on nodes based on resource availability.

---

### 2️⃣ Database Persistence

- **PostgreSQL deployed as a StatefulSet** with:
  - PersistentVolumeClaims (PVCs) for data persistence.
  - StorageClass configured for dynamic provisioning.
- **Backup strategy:** 
  - Regular scheduled backups to external storage (e.g., S3, GCS, NFS).  
  - Use tools like `pg_dump`, `wal-e`, or `Velero` for cluster-level backup.

---

### 3️⃣ Monitoring

- **Prometheus + Grafana** for metrics collection and visualization.
- **Node exporter** on nodes for hardware-level metrics.
- **Alertmanager** to notify on high CPU, memory, or pod failures.

---

### 4️⃣ Logging

- Centralized logging with **ELK stack (Elasticsearch, Logstash, Kibana)** or **Loki + Grafana**.
- Collect logs from all pods, services, and nodes.
- Enables troubleshooting and auditability.

---

### 5️⃣ CI/CD Pipeline Integration

- **Git repository triggers** → CI builds Docker images → pushes to registry → CD deploys to Kubernetes.  
- Tools: GitHub Actions, GitLab CI, Jenkins, ArgoCD, or FluxCD.
- Supports blue/green or rolling deployments for zero downtime.

---

### 6️⃣ Components Placement

| Component | Runs in Kubernetes? | Runs in VMs? | Runs outside the cluster? |
|-----------|-------------------|--------------|--------------------------|
| Application (quote-app) | ✅ Yes, in Pods (Deployment) | ❌ | ❌ |
| Database (PostgreSQL) | ✅ StatefulSet with PVC | ❌ | ❌ |
| Monitoring stack (Prometheus/Grafana) | ✅ in Pods | ❌ | ❌ |
| Logging stack (ELK/Loki) | ✅ in Pods | ❌ | ❌ |
| CI/CD runners | ❌ optional | ✅ VM or container | ❌ |
| Backup storage | ❌ | ❌ | ✅ Object storage (S3, GCS, NFS) |
| Cluster nodes / OS / kernel | ❌ | ✅ VMs or physical servers | ❌ |

---

### 7️⃣ Summary of Reasoning

- Kubernetes manages **application containers, databases, monitoring, and logging**.  
- VMs provide the **underlying nodes**, OS, and kernel isolation.  
- External storage and CI/CD runners are typically **outside the cluster**, but integrated with it.  
- High availability and resilience are ensured by:
  - Multi-node cluster  
  - Persistent storage with backups  
  - Monitoring and alerts  
  - Rolling updates via CI/CD

> This architecture ensures that applications are scalable, fault-tolerant, and production-ready.


## Secret-based Configuration

### Why is this better than plain-text configuration?

- Credentials are **not hard-coded** in deployment manifests or source code.
- Centralized management of sensitive data.
- Easier to rotate or update passwords without changing the code or redeploying manually.
- Reduces risk of accidentally leaking credentials in version control.

### Is a Secret encrypted by default? Where?

- By default, Kubernetes Secrets are **base64-encoded**, not encrypted.
- They are stored in **etcd**, the cluster’s data store.
- You can enable **encryption at rest** in etcd for stronger security, which is recommended for production clusters.
