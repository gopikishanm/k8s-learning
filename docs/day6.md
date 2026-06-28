# Day 6: Advanced Kubernetes and AI Infrastructure Concepts

## Enterprise AI on-Premise (GPUaaS)

This section details the architecture for a self-hosted GPU as a Service (GPUaaS) platform, designed to support multi-tenant enterprise AI workloads.

**Core Concepts:**
*   **LLM Inference:** This is the active phase where a trained Large Language Model (LLM) processes new user inputs (prompts) to generate real-time responses or predictions. It contrasts with the model training/fine-tuning phase.
*   **GPUaaS:** A system that allows tenants to reserve and utilize GPU resources on demand for inference, model training, and fine-tuning.

**Resource Management & Isolation:**
*   **NVIDIA Multi-Instance GPU (MIG):** This technology enables a single physical GPU to be logically partitioned into up to seven fully isolated instances. Each MIG slice is guaranteed dedicated compute cores, cache, and High-Bandwidth Memory (HBM), ensuring strong tenant isolation.
*   **Configuration:** The system involves configuring multiple GPUs: some are partitioned using MIG for multi-tenancy, while others are reserved whole for heavy, single-tenant workloads.

### System Architecture Breakdown

The platform operates across three integrated layers: Scheduling, Control, and Runtime.

1.  **Scheduling Layer (Resource Reservation):**
    *   Tenants interact with the scheduling system to view available resources (GPU, CPU, Memory).
    *   They submit a reservation request for a specific time window.
    *   Reservations move through a lifecycle: `PENDING` $\rightarrow$ `APPROVED` $\rightarrow$ `ACTIVE` $\rightarrow$ `COMPLETED`. Terminal states include `CANCELED` and `FAILED`.

2.  **Control Layer (State Management):**
    *   This layer functions as a Kubernetes-style controller, driving the system toward the desired state defined by reservations.
    *   Upon reservation approval, it provisions necessary resources and creates a dedicated namespace for the tenant.
    *   The cluster's overall state is tracked in PostgreSQL.

3.  **Runtime Layer (Execution):**
    *   When a reservation becomes active, the user accesses a pre-configured VS Code workspace.
    *   Inference engines (e.g., vLLM, Ollama, TGI, Triton) are pre-cached on the node, allowing model deployment to be a streamlined process.

## GitOps at Scale

This section covers the transition and implementation of GitOps practices for infrastructure management.

**The Challenge:**
*   A tenant migrated all repositories from Azure DevOps to GitHub. The initial plan was to use GitHub Actions for cluster deployments. However, the team predominantly relied on `helm upgrade` commands, leading to a situation where the source of truth (GitHub Action Workflow) did not guarantee reconciliation if manual changes were made directly to the cluster.

**The Solution: ArgoCD:**
*   ArgoCD was introduced to solve these consistency issues.
*   Two ApplicationSets were utilized for deployment:
    *   A standard `Application` maps a single Git repository source to a specific target cluster namespace.
    *   An `ApplicationSet` uses dynamic data inputs (generators) to automatically generate and deploy dozens or hundreds of standardized Applications simultaneously, enabling large-scale automation.

## Understanding Pod OOMKilled Events

Pod Out-of-Memory (OOMKilled) events are complex and can stem from multiple layers beyond simple CPU or memory exhaustion.

*   **Scheduling Layer:** K8s operations leading to resource preemption or eviction.
*   **Container Layer:** Linux Cgroups enforcing resource limits on the container itself.
*   **Kernel Layer:** The host system running out of global memory, triggering the kernel's OOM killer.

### OOM Behavior via Cgroups Configuration

To investigate a potential OOM event:
1.  Identify the pod UID from its metadata (`uid` field).
2.  On the host node, inspect the relevant cgroup path (e.g., `/sys/fs/cgroup/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod<pod.uid>`). This directory contains information for both the primary container and its associated pause container.

### Kernel Level OOM Analysis
*   The kernel's decision to kill a process is based on metrics like `oom_score` and `oom_score_adj`.

## References
*   [Enterprise AI On-Prem](https://sassonjoe66.medium.com/architecting-gpuaas-for-enterprise-ai-on-prem-59295ca18e96)
*   [GitOps At Scale](https://medium.com/@elad320011/gitops-at-enterprise-scale-inside-wsc-sports-deployment-platform-5ee373431309)
*   [Pod OOMKilled](https://dev.to/chunyi_wang/why-your-kubernetes-pod-was-oom-killed-and-who-really-killed-it-1jab)