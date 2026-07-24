## Day 10

### Identifying Container OS for Distroless Images

With the recent rise in container vulnerabilities, many companies are choosing to run containers with distroless images — lightweight images that don't include a shell. Without a shell, there's no risk of shell injection or interactive access. However, this makes it harder to identify which containers still use standard images.

The kernel holds all the answers. A container is simply a collection of kernel namespaces and cgroups wrapped around a regular Linux process. Every Linux process exposes a predictable path — `/proc/<PID>/root` — so reading `/proc/<PID>/root/etc/os-release` is the same as reading `/etc/os-release`. This mechanism is what tools like `nsenter`, `runc`, and other container runtimes use under the hood.

### Tracking GPU Usage

Tracking GPU usage on LLM Kubernetes clusters is tricky because there's no native GPU monitoring built in. The standard commands fall short in different ways:

- `kubectl top`: Shows only CPU and memory usage
- `nvidia-smi`: Shows per-device GPU metrics
- `nvtop` / `nvitop`: Provides a per-node view
- DCGM + Prometheus + Grafana: Delivers the complete picture, but requires significant setup

To get the full picture, you typically need a custom command. GPU wastage falls into two categories:

- **Idle:** A GPU is allocated to a pod but performs no computation and holds no meaningful memory. Utilization is zero.
- **Compute-Idle:** A GPU is pinned to a pod that has loaded a model into VRAM but isn't serving any requests. Utilization can only be measured over a rolling window.

### PCI-DSS Compliant GKE

Data governance and protection are critical when handling customer data. When dealing with PII data, you need to ensure it is stored securely and that a complete audit trail exists for every action taken against it.

A general note: CVV numbers should never be stored when handling cardholder data. Doing so is an immediate PCI violation.

When storing data, make sure it is encrypted using Customer-Managed Encryption Keys (CMEK).

Here are several strategies to minimize access to sensitive data:

- **Tokenization Vault:** Use a random string to reference a Primary Account Number (PAN) instead of storing the real value.
- **Masking:** Obscure sensitive data when displaying it in forms or logs.
- **Automated Detection:** Write a job that automatically masks or hides data before it's displayed. This removes the dependency on developers to remember the first two steps.

### References

- [Which of our Containers are Chainguard](https://breakglass.hashnode.dev/which-of-our-containers-are-chainguard)
- [One forgotten notebook on an A100. $1,800 a month](https://medium.com/@gjz140103/one-forgotten-notebook-on-an-a100-1-800-a-month-320f48a2360e)
- [PCI-DSS Compliant GKE Framework](https://blog.devops.dev/building-a-pci-dss-compliant-gke-framework-for-financial-institutions-data-protection-governance-0deaa1b72893)