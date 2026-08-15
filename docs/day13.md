## Day 13

### YouTube's Load Balancing Algorithm

Most load balancers follow a round-robin pattern, but for services like YouTube that operate at enormous scale, this approach falls short. Prequal (Probing to Reduce Queuing and Latency) is a load-balancing algorithm built for distributed, multi-tenant systems.

Instead of relying on CPU-based metrics, Prequal actively probes two per-backend signals: requests-in-flight (RIF) and recent latency.

### Etcd Crashes

Karmada (Kubernetes Armada) is a management system that lets you run cloud-native applications across multiple Kubernetes clusters and clouds without changing your applications. Karmada itself uses etcd to store its state.

Karmada pods were crashing every few minutes. The logs pointed to etcd timing out. etcd's consistency guarantees come at a cost: it uses a write-ahead log and relies on `fsync` calls completing within tight time windows. When storage is slow, etcd misses its internal heartbeat and election deadlines, causing leader election to fail and the cluster to lose quorum. The fix was to optimize the underlying ZFS filesystem.

The following parameters were tuned:

- **sync=disabled** — ZFS normally waits for data to be physically written to disk before confirming a write to the application (synchronous writes). Disabling sync means ZFS acknowledges writes immediately, which dramatically reduces write latency.
- **compression=lz4** — Enables transparent compression on all data using the lz4 algorithm.
- **atime=off** — By default, filesystems record the last access time on every file every time it is read. Disabling atime means reads are just reads. Almost every performance-tuned Linux system does this.
- **recordsize=8k** — ZFS uses a configurable record size (default 128K) when writing data. Setting it to 8K aligns ZFS's I/O unit closer to what etcd actually does, reducing write amplification. Instead of reading and rewriting a 128K block to change a few bytes, ZFS only touches an 8K block.

### Name Resolution Issue

When DNS resolution failures occur in a Kubernetes cluster, the first component to check is CoreDNS. Look for resource limits or configuration issues.

If the problem is resource-related, the recommended approach is to scale CoreDNS pods using the cluster proportional autoscaler rather than the Horizontal Pod Autoscaler (HPA). CoreDNS throughput depends more on request rate and memory than on CPU.

If CoreDNS looks healthy, check the `resolv.conf` configuration inside the pods. By default, Kubernetes sets `ndots:5` in every pod's `/etc/resolv.conf`. The `ndots` option sets the threshold for how many dots a domain name must contain before it is treated as a fully qualified domain name (FQDN) rather than a relative name. On most standard Linux systems, the default value is 1.

Kubernetes's `ndots:5` setting carries a performance penalty. When a pod queries a service name with few dots, the system cycles through the local search path and generates multiple wasted DNS queries. As a best practice, set `ndots` based on what the microservices on your cluster actually require.

### References

- [YouTube’s Load Balancing Algorithm](https://medium.com/@sathwick.p7/how-i-rebuilt-youtubes-load-balancing-algorithm-in-go-9a8ea8b39c8f)
- [Etcd Crashes](https://nubificus.co.uk/blog/etcd/)
- [Name Resolution Issue](https://medium.com/@a.warkhade98/the-issue-159436266391)
- 