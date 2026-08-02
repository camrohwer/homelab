# Secret Management

## Overview
This repository uses **SOPS** with **age encryption** to manage Kubernetes secrets.

Encrypted secrets are stored in Git. During cluster deployment, Ansible running on the deployment controller decrypts the secrets using a private age key and applies them to Kubernetes.

Applications deployed by ArgoCD consume the already-created Kubernetes Secrets.

This repository intentionally uses Ansible for secret decryption instead of an ArgoCD secret decryption plugin. ArgoCD is responsible for application deployment, while Ansible is responsible for securely injecting secrets into the cluster during provisioning.

The workflow is:

```
Developer machine 
    |
    | Encrypt with SOPS + age
    |
    v
Encrypted Secret YAML
    |
    | Git commit
    |
    v
Ansible deployment controller
    |
    | Uses private age key
    |
    | Decrypts with SOPS
    |
    | Applies Secret manifest
    v 
Kubernetes API Server
    |
    v
Kubernetes Secret
    |
    v
ArgoCD applications consume Secret
    |
    v
Application
```

The repository contains:
- Encrypted secret manifests
- The SOPS configuration file
- The public age encryption key

The private age key is only stored on the Ansible deployment controller.

The private key is required to decrypt secrets and must never be committed.

---

## Why SOPS + age?

Traditional K8s secrets are only base64 encoded.

SOPS provides:

- Encryption of secret values
- Git friendly encypted YAML
- Support for age
- Safe storage of encrypted secrets in source control

The repository can be public without exposing credetials, which is helpful for easy setup of ArgoCD.

---

## Secret Deployment Architecture

Secrets are intentionally applied before application deployment.

The deployment order is:

1. Bootstrap K3s cluster
2. Install ArgoCD
3. Create required namespaces
4. Decrypt and apply SOPS secrets with Ansible
5. Deploy applications through ArgoCD
6. Applications consume K8s Secrets

---

## Encryption Configuration

The `.sops.yaml` file in the project root defines encryption rules.

```yaml
creation_rules:
  - path_regex: gitops/secrets/.*\.yaml
    encrypted_regex: '^(data|stringData)$'
    age: agexxxxxxxxxxxxxxxx
```

This tells SOPS:
- Encrypt files inside `gitops/secrets`
- Only encrypt K8s secret values
- Use the specified age public key

The public key is safe to commit.

The private key was generated locally and is stored at:

`~/.config/sops/age/keys.txt`

This file must NEVER be commited.

The key was generated using: `age-keygen -o ~/.config/sops/age/keys.txt`

---

## Applying Secrets During Deployment

Secrets are automatically decrypted and applied by Ansible.

The playbook `apply-secrets.yaml` does the following:

1. Finds encrypted secret files in `gitops/secrets/*.enc.yaml`
2. Decrypts them using SOPS where `SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt`
3. Applies the resulting K8s Secret objects.

```
gtiops/secrets/grafana-admin.enc.yaml
    |
    v
sops decrypt
    |
    v
Secret/grafana-admin
    |
    v
Grafana Helm deployment
```
The plaintext secret only exists temporarily during deployment.

## Creating a new secret

### 1. Create the Kubernetes Secret manifest

Example:

`gitops/secrets/example.yaml`

Before encryption:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: example-secret
  namespace: example

type: Opaque

stringData:
  username: admin
  password: mypassword
```

---

### 2. Encrypt the secret

Run:

```bash
sops encrypt -i gitops/secrets/example.yaml
```
```bash
mv gitops/secrets/example.yaml gitops/secrets/example.enc.yaml
```

The file is now encrypted:

```yaml
stringData:
  username: ENC[AES256_GCM,...]
  password: ENC[AES256_GCM,...]
```

The plaintext values are no longer stored.

--- 

### 3. Verify the secret encryption

Inspect the file to verify encryption and test decryption with:

```bash
sops decrypt gitops/secrets/example.enc.yaml
```

---

## Updating an Existing Secret

To modify an existing secret, use SOPS directly:

```bash
sops gitops/secrets/example.enc.yaml
```

SOPS will:
1. Decrypt the file into memory.
2. Open it in vim
3. Re-encrypt it automatically when you save and exit.

---

## Requirements

The follow must be installed on the deployment controller:

- `sops`
- `age`


