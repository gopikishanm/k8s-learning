## Day 8

### Throttling-Induced Contention Amplification

This article explains how to troubleshoot API latency in Python APIs when P50 is stable but P99 variance is very high. The approach applies to many programming languages, including those with Python's GIL, JVM monitors, Ruby's GVL, and Node.js event loop.

**Key terms**

1. **GIL**: Python's Global Interpreter Lock — allows only one thread of bytecode to execute at a time.
2. **CFS**: Completely Fair Scheduler (default 100ms window).

**The problem**

A shared lock is held by thread T, which the OS scheduler then preempts due to resource exhaustion. All other threads waiting on the shared lock remain blocked during the throttle interval. This causes latency spikes in the API.

**Possible solutions**

- Increase CPU requests for the pod to reduce the CPU burstable zone.
- Match requests to limits to eliminate the problem entirely.
- Scale horizontally.

### Feedback Loops Behind Kubernetes

This article discusses operators — a feedback controller pattern. It uses PostgreSQL as an example to show the stages involved in running highly available, reliable applications at scale.

When you run an application across multiple nodes and want to ensure everything works as expected, you start with a **watchdog script** to monitor nodes and alert you on failures. This script runs as a continuous loop, watching the infrastructure. Suppose you want to track disk usage and alert when it reaches 80%. You update the script and run it. As the script runs continuously, it feeds the current output into the next iteration and checks the results. This is called a **closed feedback loop**.

In a closed feedback loop, you can add multiple conditions and get notified for all failures.

There is another concept: **open-loop** control. This involves a user logging into a node, running an action, and assuming it **works**.

**The loops used by Kubernetes**

- **Spinning up a container**: The kubelet's desired state is the set of containers assigned to the node by the API server. The observed state is the actual pods running on the node. When there is a mismatch, the kubelet tries to spin up pods.
- **Picking a node**: The scheduler watches for pods with no node assigned, records the placement, and the kubelet picks it up.
- **Attaching the disk**: CSI and PVC sync. Cloud providers develop their own Container Storage Interfaces to handle disk creation, resizing, and deletion.
- **Making them find each other**: CNIs (Container Network Interfaces).

In Kubernetes, the watchdog script is called a **controller**. The standard way to build one in Go is **controller-runtime**, which has a core function called **Reconcile**.

**Two ways to build any closed loop**

- **Edge-triggered notifications**: Act on events — disk usage crossed 80%, replicas increased to 2.
- **Level-triggered**: Act on current state — pod missing, disk is at 85%.

Kubernetes controllers combine both: edge-triggered notifications with level-triggered logic.

**Informer**: Opens a single watch against the API server for a given resource type and streams every event on that resource. These events are placed into **work queues**.

Since the informer streams many events, all of them are written to the informer's local cache. This is how a controller reconciles thousands of objects without falling over. One common issue with caching is that the data read might be stale. Eventual consistency is assumed since the loop runs frequently.

**Other terms**: Self Healing by Design, Expectations Pattern, Cascade Control, coalescing delay, resyncer, leader election.

### Code Mode

This article explains how to reduce token usage when we use MCP.

### GPU Utilization

This article explains how GPU's are used in inference.

### References

- [Throttling-Induced Contention Amplification](https://medium.com/@prashant_pathak/you-dont-have-a-gil-problem-you-have-a-cpu-problem-24deeadfea4a)
- [Feedback Loops Behind Kubernetes](https://planetscale.com/blog/the-feedback-loops-behind-kubernetes)
- [Control Theory](https://en.wikipedia.org/wiki/Control_theory)
- [Kubernetes Controllers](https://ahmet.im/blog/controller-pitfalls/)
- [Evicting MCP Tool Calls](https://dev.to/mikhae1/evicting-mcp-tool-calls-from-your-kubernetes-cluster-428k)
- [Code Mode](https://blog.cloudflare.com/code-mode/)
- [GPU Utilization](https://medium.com/google-cloud/what-does-4-4-gpu-utilization-actually-mean-ee61fabebbf0)