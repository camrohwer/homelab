# Networking

## DNS Resolution

Internal DNS is provided by Pi-hole.

Applications are assigned DNS names:

| Service | DNS Name | IP |
|---|---|---|
| Grafana | grafana.homelab.local | 192.168.20.10 |
| Prometheus | prometheus.homelab.local | 192.168.20.10 |
| ArgoCD | argocd.homelab.local | 192.168.20.10 |

Pi-hole resolves these names to the K3s master node.

## Traffic Flow

Client:

Browser
    |
    v
Pi-hole DNS
    |
    v
192.168.20.10
    |
    v
Traefik Ingress
    |
    v
Kubernetes Service
    |
    v
Pod

## Design Decision

Services do not receive static IP addresses.

DNS points to the ingress layer. Kubernetes handles service discovery internally.
