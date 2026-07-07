# Homelab k3s Runbook

## 1. Test SSH connectivity for inventory
```bash
ansible -i inventory.yaml all -m ping
```

## 2. Run initial bootstrap
```bash
ansible-playbook -i inventory.yaml bootstrap.yaml
```

## 3. Configure kubectl 
```bash
ansible-playbook -i inventory.yaml configure-kubectl.yaml
```

## Hard Reset
```bash
ansible-playbook -i inventory.yaml reset-pis.yaml
```
