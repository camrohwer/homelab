# Homelab TLS Certificates

## Overview

This document describes how HTTPS certificates are managed in my homelab Kubernetes cluster.

The goal is to have all internal services accessible through HTTPS using a private Certificate Authority (CA), while keeping certificate issuance and renewal fully managed through GitOps.

The certificate flow is:

```
User Device
    |
    | trusts homelab-root-ca.crt
    |
    v
Traefik Ingress Controller
    |
    | uses TLS secret
    |
    v
Kubernetes Service
```

Certificates are issued by cert-manager using an internal CA.

```
cert-manager
      |
      v
ClusterIssuer
      |
      v
homelab-root-ca
      |
      +----------------+
      |                |
      v                v
grafana-tls       prometheus-tls
      |
      v
Traefik HTTPS ingress
```

---

# Components

## cert-manager

cert-manager manages certificate resources inside Kubernetes.

Responsibilities:

- Request certificates
- Renew certificates before expiration
- Store certificates as Kubernetes TLS secrets
- Provide certificates to ingress controllers

---

## Internal Certificate Authority

The homelab uses a private CA.

The CA certificate is:

```
homelab-root-ca.crt
```

This certificate must be trusted by client machines that access internal services.

The CA signs certificates for:

* Grafana
* Prometheus
* ArgoCD
* Future internal applications

---

## Traefik

Traefik is the TLS termination point.

The flow is:

```
Browser
   |
   | HTTPS
   |
Traefik
   |
   | HTTP
   |
Application
```

Applications do not manage TLS themselves.

---

# Initial Certificate Setup

## 1. Install cert-manager

cert-manager is deployed through ArgoCD.

The deployment order is:

```
ArgoCD
 |
 v
cert-manager
 |
 v
ClusterIssuer
 |
 v
Certificates
 |
 v
Ingress TLS
```

---

## 2. Create the Root CA

The root CA secret is stored in Kubernetes:

```
namespace: cert-manager

secret:
  homelab-root-ca
```

The CA contains:

```
ca.crt
tls.crt
tls.key
```

The CA is used by the ClusterIssuer to sign service certificates.

---

## 3. Create the ClusterIssuer

The ClusterIssuer tells cert-manager how to issue certificates.

Example:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer

metadata:
  name: homelab-ca

spec:
  ca:
    secretName: homelab-root-ca
```

This allows certificates to request:

```yaml
issuerRef:
  name: homelab-ca
  kind: ClusterIssuer
```

---

# Installing the CA Certificate on Arch Linux

The CA certificate must be installed on client machines.

Export the CA certificate:

```bash
kubectl get secret homelab-root-ca \
  -n cert-manager \
  -o jsonpath='{.data.ca\.crt}' \
  | base64 -d > homelab-root-ca.crt
```

Copy it into the Arch certificate trust store:

```bash
sudo cp homelab-root-ca.crt /etc/ca-certificates/trust-source/anchors/
```

Update the trust database:

```bash
sudo trust extract-compat
```

Verify:

```bash
trust list | grep homelab
```

After this, command line tools and browsers using the system trust store will trust certificates issued by the homelab CA.

---

# DNS Setup with Pi-hole

Internal services use the `.local` domain:

Examples:

```
grafana.homelab.local
prometheus.homelab.local
argocd.homelab.local
```

Pi-hole provides internal DNS resolution.

Example:

```
grafana.homelab.local
        |
        v
192.168.20.10 (Master)
```

The request flow becomes:

```
Browser
   |
   | DNS lookup
   |
Pi-hole
   |
   | returns Traefik IP
   |
Traefik
   |
   | TLS termination
   |
Grafana
```

---

# Adding a Certificate for a New Application

When adding a new application, follow this process.

Example application:

```
app.homelab.local
```

---

## 1. Create the Certificate Resource

Create:

```
gitops/infrastructure/cert-manager/certificates/app.yaml
```

Example:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate

metadata:
  name: app
  namespace: app-namespace

spec:
  secretName: app-tls

  dnsNames:
    - app.homelab.local

  issuerRef:
    name: homelab-ca
    kind: ClusterIssuer
```

Commit and allow ArgoCD to sync.

---

## 2. Verify Certificate Issuance

Check:

```bash
kubectl get certificate -n app-namespace
```

Expected:

```
NAME   READY   SECRET
app    True    app-tls
```

The TLS secret should now exist:

```bash
kubectl get secret app-tls -n app-namespace
```

---

## 3. Add the Ingress TLS Configuration

Create or update the ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: app
  namespace: app-namespace

spec:
  ingressClassName: traefik

  tls:
    - hosts:
        - app.homelab.local
      secretName: app-tls

  rules:
    - host: app.homelab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app
                port:
                  number: 80
```

The important relationship is:

```
Certificate
    |
    v
Secret
    |
    v
Ingress
    |
    v
Traefik
```

The `secretName` must match in both resources.

---

## 4. Add DNS Entry

Add a Pi-hole local DNS record:

```
app.homelab.local
```

pointing to:

```
192.168.20.10
```

(or the IP address of the Traefik load balancer).

---

# Operational Model

The desired workflow is:

```
Create application
        |
        v
Create Certificate resource
        |
        v
cert-manager issues certificate
        |
        v
Create Ingress referencing TLS secret
        |
        v
Add Pi-hole DNS record
        |
        v
Access application over HTTPS
```

No manual certificate generation should be required after the initial CA setup.

