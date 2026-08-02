# Secret Management

## Overview

This repository is using **SOPS** with **age encryption** to manage K8s secrets in GitOps.

Secrets are stored in Git as encrypted YAML files. The actual secret values are never commited in plaintext.

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
    v ArgoCD
    |
    | Decrypts secret
    |
    v 
Kubernetes Secret
    |
    v
Application
```

The repository contains encrypted secrets and the public encryption key. The private age key is kept outside the repository.

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

## Encryption Configuration

The `.sops.yaml` file in the project root defines encryption rules.

```
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

## Creating a new secret

### 1. Create the Kubernetes Secret manifest

Example:

`gitops/secrets/example.yaml`

Before encryption:

```
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

`sops encrypt -i gitops/secrets/example.yaml`

The file is now encrypted:

```
stringData:
  username: ENC[AES256_GCM,...]
  password: ENC[AES256_GCM,...]
```

The plaintext values are no longer stored.

--- ### 3. Verify the secret encryption

Inspect the file to verify encryption and test decryption with:

`sops decrypt gitops/secrets/example.yaml`






