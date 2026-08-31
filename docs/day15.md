## Day 15

### Emerging Tools in Kubernetes Community

Kubernetes Resource Orchestrator(KRO) bundles multiple Kubernetes resources into a single, reusable API. The base custom resource is `ResourceGraphDefinition` which allows to define kubernetes core resources in single spec. We can define `deployment`, `service` and `ingress` as part of this custom resource.

OpenTelemetry is open standard for collecting logs, metrics and traces. Currently, prometheus stack is used for metrics which fluent-bit is go to tool for logging.

Kueue is used for intelligent queueing for batch and GPU-intensive workloads.

### Kafka on Kubernetes

When we deploy kafka on kubernetes, application tuning might solve only some lag issues. The root cause can be due to OS or Kubernetes behaviour.

The problem was that brokers were doing consistent disk reads. Due to high throughput requirements this was a red flag. 

Kafka relies on OS page cache. In healthy cluster, all the consumer reads should occur from memory rather than disk reads. With all application changes intact when moving from EC2 to EKS, the only changes in stack were OS migration to AL 2023 and cgroup v1 to cggroup v2.

`vmscan` is used to track kernel virtual memory operations. The tool projected that kernel was aggresively reclaiming the page cache. This is cgroup v2 behaviour and specific to `memory.high`. Once a workload crosses the high memory threshold, the kernel can start applying reclaim pressure inside that cgroup even if the node still has free memory available.

The fix was done in 2 steps

- Remove pod memory limits
- Tune `vm.min_free_kbytes`.

### References

- [Emerging Tools in Kubernetes Community](https://kube.today/emerging-tools-shaping-kubernetes-future)
- [Kafka on Kubernetes](https://dev.to/yaakovamar/kafka-on-kubernetes-performance-lessons-for-any-disk-heavy-data-service-3bl5)