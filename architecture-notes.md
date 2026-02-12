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
