# k3s Homelab Ansible Playbook

Provisions a highly-available k3s cluster using embedded etcd.

## Cluster layout

| Role           | IP          | Hostname      |
|----------------|-------------|---------------|
| Initial master | 10.0.20.15  | k3s-master-1  |
| Master #2      | 10.0.20.17  | k3s-master-2  |
| Master #3      | 10.0.20.19  | k3s-master-3  |

> **Note:** `inventory/hosts.yml` has an empty `workers` group — add dedicated
> worker IPs there if you add agent-only nodes later.

## Prerequisites

- Ansible ≥ 2.12 with the following collections:
  ```
  ansible-galaxy collection install ansible.posix community.general
  ```
- SSH access to all nodes from the control machine (key-based recommended)
- Python 3 on all target nodes

## Quick start

```bash
# 1. Generate a strong cluster token (do this once and keep it safe)
export K3S_TOKEN=$(openssl rand -hex 32)

# 2. Run the full playbook
ansible-playbook -i inventory/hosts.yml site.yml -e "k3s_token=${K3S_TOKEN}"

# 3. Use the fetched kubeconfig
export KUBECONFIG=./kubeconfig-k3s-master-1.yaml
kubectl get nodes -o wide
```

## Variables

| Variable | Default | Description |
|---|---|---|
| `k3s_version` | `v1.29.4+k3s1` | k3s release to install |
| `k3s_token` | `changeme-...` | Pre-shared cluster secret |
| `k3s_cluster_cidr` | `10.42.0.0/16` | Pod network CIDR |
| `k3s_service_cidr` | `10.43.0.0/16` | Service network CIDR |
| `k3s_flannel_backend` | `vxlan` | Flannel backend |
| `k3s_disable_components` | `[servicelb]` | Components to skip |
| `k3s_server_extra_args` | `""` | Extra `k3s server` flags |
| `k3s_agent_extra_args` | `""` | Extra `k3s agent` flags |

Override any variable with `-e` on the command line or in an Ansible Vault file.

## Selective runs (tags)

```bash
ansible-playbook -i inventory/hosts.yml site.yml --tags prereqs   # system prep only
ansible-playbook -i inventory/hosts.yml site.yml --tags masters    # masters only
ansible-playbook -i inventory/hosts.yml site.yml --tags workers    # workers only
```

## Directory structure

```
k3s-ansible/
├── inventory/
│   └── hosts.yml              # Cluster inventory
├── group_vars/
│   ├── all.yml                # Shared variables
│   ├── masters.yml            # Master-specific variables
│   └── workers.yml            # Worker-specific variables
├── roles/
│   ├── prereqs/               # System hardening, sysctl, firewall
│   │   └── tasks/main.yml
│   ├── k3s-master/            # k3s server install & kubeconfig fetch
│   │   ├── handlers/main.yml
│   │   ├── tasks/main.yml
│   │   └── templates/k3s-config.yaml.j2
│   └── k3s-worker/            # k3s agent install
│       └── tasks/main.yml
└── site.yml                   # Main entry-point playbook
```
