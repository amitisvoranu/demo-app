## Security
- As CISO I want that all deployment manifests that are being generated are running in their own isolated network through Kubernetes policy
- As CISO I want all pods to have a read-only file system

## Infrastructure
- As a Solutions Architect I want to use type 'LoadBalancer' for all my k8s services
- As a Solutions Architect I want the following annotations to be added to all my ingress resources:
    - kubernetes.io/ingress.class: alb
    - alb.ingress.kubernetes.io/scheme: internet-facing
    - alb.ingress.kubernetes.io/target-type: instance