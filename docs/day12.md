## Day 12

### OSS Kubernetes Security Console

Security related signals come from multiple sources in a running kubernetes cluster.

- Falco sees runtime behaviour
- Trivy sees vulnerable images 
- kubescape captures posture and compliance drift
- kyverno helps with policy enforcements

When a shell is spawned in container, instead of panicking we can understand how the pod is running to ensure we have proper context on the next actions. The shell might be used for debugging or one time action. 

The goal is not to replace existing systems, rather it is to make open source security signals queryable using mcp server.

### nodes/proxy GET RBAC permission

nodes/proxy GET RBAC permission allows any ServiceAccount to execute code inside any Pod in the cluster, without leaving a single trace in the audit logs. This is dangerous.

Steps to get unlimited access 

- kubectl exec uses websocket connection whose handshake is HTTP GET
- kubelet maps this to GET RBAC verb
- It checks nodes/proxy GET and authorizes operation
- No secondary check is performed for CREATE verb

Official kubernetes status is that this won't be fixed. 

The preventive measures suggested are 

- Fine grained API authorization with no reference to nodes/proxy GET
- Blocking access to kubelet port 10250 from components that don't need the access
- Blocking creation of new roles which depend on nodes/proxy
- Monitoring the audit log for SubjectAccessReviews

### References

- [OSS Kubernetes Security Console](https://cloudsecburrito.com/building-an-oss-kubernetes-security-console-with-mcp/)
- [nodes/proxy GET RBAC permission](https://blog.zwindler.fr/en/2026/05/19/nodes/proxy-get-one-kubernetes-permission-too-many/)