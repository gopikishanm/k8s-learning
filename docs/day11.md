## Day 11

### AI SRE

Claude Code Channels is a plugin-based feature that lets you send messages from Telegram, Discord, or iMessage directly into a running Claude Code session on your local machine.

Imagine pushing an alert into a Claude session that has access to `kubectl`, Git (to inspect code), and the ability to run runbooks against your cluster. That gives you an AI-powered SRE.

Sending thousands of alerts to a single Claude session would quickly fill up its context window. To work around this, the main agent only delegates alerts — it never acts on them directly. For each alert that needs investigation, the main agent spawns a sub-agent (a separate Claude instance) with its own context window. The sub-agent handles the heavy lifting, and once the issue is resolved, its context is discarded.

One of the core challenges is instructing the agent on which operations are safe and which are not.

**Safe Operations**

- Restart a deployment
- Delete a stuck pod
- Clear a stuck workflow
- Force refresh an ArgoCD application

**Unsafe Operations**

- Deleting an ArgoCD application
- Deleting any Kubernetes resources (namespace, PVC, etc.)
- Deleting databases
- Pushing to Git
- Scaling to zero

**Always escalate to humans for:**

- Database errors
- Certificate or mTLS failures
- Anything the agent does not understand or that requires human intervention

At the Kubernetes level, RBAC is used to restrict what actions an AI agent can perform.

### GPU Bill Analysis

This article covers several ways to analyze GPU clusters and track GPU spending.

- Always taint GPU nodes so non-GPU workloads cannot schedule on them.
- Apply namespace-level `ResourceQuotas`.
- Use Dynamic Resource Allocation (DRA), available from Kubernetes 1.32.
- Define SLOs that measure inference performance:
  - **Time to First Token (TTFT)**
  - **Tokens per Second (TPS)**
- Ensure all workloads have relevant cost labels.

The platform team is responsible for making sure all of the above are followed. Enforcing strict rules with Gatekeeper or Kyverno policies helps keep things on track.

### References

- [AI SRE](https://medium.com/@leo_62530/were-a-3-person-tech-team-running-production-kubernetes-so-we-built-an-ai-sre-61ee28810448)
- [GPU Bill Analysis](https://medium.com/@mateenanjum/the-gpu-bill-was-40-000-nobody-knew-why-e6e953b25f4a)