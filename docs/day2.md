## Day 2

### Article 1

The article describes how we can use EKS Auto Mode to setup load balancer with TLS encrypted enabled using certificate stored in AWS Cert Manager.

Prerequisites:

- EKS Automode cluster
- Domain name registed on Route53
- Certificate in AWS Certificate Manager
- VPC Subnet Tagging - Enable EKS to discover these subnets
- Public Subnets: kubernetes.io/role/elb = 1
- Private Subnets: kubernetes.io/role/internal-elb = 1
- IAM Permissions: IAM Role must have AmazonEKSLoadBalancingPolicy. This allows cluster to provision ALB on our behalf

We have to define 2 resources - IngressClassParams and IngressClass. IngressClass has controller details i.e 'controller: eks.amazonaws.com/alb'

We use annotation 'alb.ingress.kubernetes.io/certificate-arn:' to point to certificate. We can integrate ingress resource with external dns for auto creation of record on Route53.


### References

- https://medium.com/@dicksonvictoromasi/from-acm-to-alb-a-step-by-step-guide-to-managed-tls-ssl-in-eks-auto-mode-ca66cc0da72f