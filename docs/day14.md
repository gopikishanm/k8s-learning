## Day 14

### End to End Observability of Workloads

In the Kubernetes ecosystem, several tools help monitor workloads:

- **Prometheus stack** for metrics
- **Fluentd / Fluent-bit** for logging
- **Tracing tools** for request tracking
- **Kubernetes itself** adds infrastructure events, pod evictions, node pressure, and more

To combine all these sources into a single, centralized view, you need a common `selector` that links them together. One approach is to add metadata labels to pod environments and let OpenTelemetry read their values.

**The label cardinality issue:** When choosing a selector, make sure its values don't change frequently. For example, using the `pod id` as a selector creates a unique value for every pod. If your workloads scale up and down aggressively, this generates a huge number of metrics and can destabilize Prometheus. The key lesson is that metric labels should be bounded. Instead, rely on labels you control — like namespace, deployment name, service name, and environment — since these don't produce unbounded unique values.

Another decision worth making is to ship Kubernetes event logs to the same destination as your application logs using `kube-event-exporter`. This makes it easier to diagnose issues — for instance, linking a scale-down event to a spike in API latency.

### KEDA — Event Driven Autoscaling

For general workloads, we typically monitor CPU and memory metrics and set up Horizontal Pod Autoscalers (HPAs) to scale replicas up or down. But resource metrics alone don't always address the root problem.

**KEDA** (Kubernetes Event-Driven Autoscaling) helps you scale based on external event sources instead of relying only on CPU and memory.

KEDA serves two roles:

- **Metrics adapter:** Exposes external metrics from sources like Kafka or RabbitMQ to the Kubernetes metrics API.
- **Operator / controller:** Watches the `ScaledObject` custom resource and drives the HPA — something the native HPA cannot do on its own.

Modern application bottlenecks often live in:

- Message queue depth
- Database connection pool
- Cache misses
- HTTP request length
- Cron-based jobs

KEDA introduces three custom resources: `ScaledObject`, `ScaledJob`, and `TriggerAuthentication` / `ClusterTriggerAuthentication`.

### Outage from a Dead NLB

Every runtime has a different DNS cache TTL. In the JVM, when the security manager is enabled, the default cache setting is effectively forever — the JVM resolves the IP addresses of Network Load Balancers (NLBs) only at startup and caches them indefinitely.

The fix is to configure the following values:

For reference, here is the Java security configuration that overrides the default DNS caching behavior:

```
# $JAVA_HOME/conf/security/java.security
networkaddress.cache.ttl=30
networkaddress.cache.negative.ttl=5
```

This forces the JVM to re-resolve DNS every 30 seconds, with a 5-second timeout for failed lookups. The trade-offs are:

- More DNS queries are generated
- A 30-second window still exists before failover takes effect
- The setting must be applied per runtime instance

### References

- [End to End Observability of Workloads](https://hackernoon.com/engineering-end-to-end-observability-for-kubernetes-workloads)
- [KEDA - Event Driven Autoscaling](https://the-devops-engineer.medium.com/kubernetes-keda-autoscaling-scale-smarter-not-harder-d186da29175a)
- [Outage from a dean NLB](https://dev.to/claire_nguyen/an-8-minute-outage-from-a-dead-nlb-and-a-jvm-that-cached-dns-forever-4dmj)

### To Revisit

- [Volcano Controllers](https://medium.com/@myown4500/inside-volcano-controllers-gang-scheduling-state-machines-and-real-kubernetes-logs-07b32a21c504)