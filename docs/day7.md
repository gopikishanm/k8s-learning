## Day 7 

### Cilium Network Policies L7 HTTP Filtering

Kubernetes Network Policies only support L3/L4 filtering (IP addresses and ports). Cilium uses eBPF to provide L7 protocol-aware policies. This lets you filter traffic by HTTP paths, methods, and headers, all enforced in the Linux kernel without sidecars.

eBPF (Extended Berkeley Packet Filter) is a Linux kernel technology that lets you run custom programs directly inside the kernel.

Cilium's eBPF datapath removes iptables complexity.

### Chaos Mesh

A Chaos engineering platform that allows to simulate failures in environments and helps measure the impact.

### References

- [Cilium Network Policies](https://medium.com/pickme-engineering-blog/cilium-network-policies-l7-http-filtering-with-ebpf-on-kubernetes-cfddeb8434cb)
- [Chaos Mesh](https://tech.groww.in/taming-the-storm-building-growws-internal-chaos-engineering-platform-5269d65260aa)