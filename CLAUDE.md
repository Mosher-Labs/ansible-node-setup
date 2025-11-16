# Mosher Labs Ansible Node Setup - Project Memory

This file contains persistent context for Claude Code sessions on this project.
It will be automatically loaded at the start of every session.

## Project Overview

This repository contains Ansible playbooks and roles for provisioning and
configuring a 3-node k3s Kubernetes cluster. This is the **foundation layer**
that sets up the cluster before GitOps (ArgoCD) takes over.

**Key Details:**

- **Purpose:** Automated k3s cluster provisioning
- **Cluster Type:** 3-node k3s (1 server + 2 agents)
- **OS:** Ubuntu Server
- **Ansible User:** `ansible` (with sudo NOPASSWD)
- **SSH Key:** `~/.ssh/ansible_key` (stored in 1Password)
- **Network:** 192.168.87.x subnet

## Cluster Topology

### Server Node (Control Plane)

- **Hostname:** `hp-elitedesk`
- **IP:** 192.168.87.10
- **Role:** k3s server (control plane + workload)
- **Inventory Group:** `[server]`

### Agent Nodes (Workers)

- **macmini-2010:** IP 192.168.87.11, Inventory Group `[agents]`
- **battlestation-ubuntu:** IP 192.168.87.12, Inventory Group `[agents]`

### Legacy/Commented Nodes

Previously used VirtualBox test nodes (commented out in inventory):

- server-node: 192.168.202.14
- agent-node1: 192.168.202.13
- agent-node2: 192.168.202.15

## Repository Structure

```text
ansible-node-setup/
├── ansible.cfg              # Ansible configuration
├── inventory.ini            # Cluster node inventory
├── playbook.yml             # Main playbook
├── requirements.yml         # Ansible Galaxy dependencies
├── roles/
│   ├── dependencies/        # System dependencies (all nodes)
│   ├── install_k3s_server/  # k3s server setup
│   └── install_k3s_agents/  # k3s agent setup
├── .ansible/                # Ansible runtime data
└── .github/                 # CI/CD workflows
```

## Playbook Workflow

The main playbook (`playbook.yml`) runs in this order:

1. **Setup dependencies** (all nodes)
  - System updates
  - Required packages
  - Kernel optimizations
  - Container runtime prerequisites

1. **Install k3s server** (server group)
  - Downloads and installs k3s
  - Configures as control plane
  - Generates node token for agents
  - Sets up kubeconfig

1. **Install k3s agents** (agents group)
  - Downloads and installs k3s
  - Joins cluster using server token
  - Registers as worker nodes

## Initial Node Setup (One-time per node)

### Base OS Installation

1. Install Ubuntu Server (minimal installation)
1. Set hostname during install or via `hostnamectl`
1. Configure static IP or DHCP reservation

### Create Ansible User

On each node after base install:

```bash
ssh ubuntu-virtualbox  # or node hostname
sudo adduser ansible
sudo usermod -aG sudo ansible
sudo su
echo 'ansible ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
exit
exit
```

### Copy SSH Key

On your local machine:

```bash
# Copy SSH public key to node
ssh-copy-id ansible@<node-hostname>

# Export private key from 1Password
op item get "k3s-node-ansible-ssh-key" --field "private key" > $HOME/.ssh/ansible_key
chmod 600 $HOME/.ssh/ansible_key
```

## Running the Playbook

### Prerequisites

```bash
# Install mise (version manager)
mise install pipx

# Install Ansible via pipx
pipx install --include-deps ansible
pipx ensurepath

# Install Ansible Galaxy requirements
ansible-galaxy install -r requirements.yml
```

### Execute Playbook

```bash
# Run full playbook (all nodes)
ansible-playbook -i inventory.ini playbook.yml

# Run specific role
ansible-playbook -i inventory.ini playbook.yml --tags dependencies

# Run against specific host
ansible-playbook -i inventory.ini playbook.yml --limit hp-elitedesk

# Dry-run (check mode)
ansible-playbook -i inventory.ini playbook.yml --check

# Verbose output for debugging
ansible-playbook -i inventory.ini playbook.yml -vvv
```

## Kubeconfig Setup

After k3s server is installed, get the kubeconfig:

```bash
# Copy kubeconfig from server
scp -i ~/.ssh/ansible_key ansible@hp-elitedesk:/etc/rancher/k3s/k3s.yaml ~/k3s.yaml

# Update server IP (replace 127.0.0.1 with actual IP)
sed -i '' 's/127.0.0.1/192.168.87.10/g' ~/k3s.yaml

# Set permissions
chmod 600 ~/k3s.yaml

# Use kubeconfig
export KUBECONFIG=~/k3s.yaml

# Verify cluster
kubectl get nodes
kubectl get deployments
kubectl get pods -A
kubectl get services -A
```

## Ansible Configuration

### ansible.cfg

- Inventory: `inventory.ini`
- Host key checking: Disabled (for homelab convenience)
- SSH args: ControlMaster for connection reuse
- Fact caching: JSON (local)

### Inventory Variables

Set in `inventory.ini` under `[all:vars]`:

- `ansible_user=ansible` - SSH user
- `ansible_ssh_private_key_file=~/.ssh/ansible_key` - SSH key path

## Common Ansible Commands

```bash
# Ping all nodes
ansible all -i inventory.ini -m ping

# Check connectivity
ansible all -i inventory.ini -m setup

# Run ad-hoc command on all nodes
ansible all -i inventory.ini -a "uptime"

# Run with sudo
ansible all -i inventory.ini -b -a "apt update"

# Gather facts from server node
ansible server -i inventory.ini -m setup
```

## k3s Specific Details

### Server Installation

- **Install script:** `https://get.k3s.io`
- **Service:** `k3s.service`
- **Kubeconfig:** `/etc/rancher/k3s/k3s.yaml`
- **Token:** `/var/lib/rancher/k3s/server/node-token`
- **Data dir:** `/var/lib/rancher/k3s`

### Agent Installation

- **Install script:** `https://get.k3s.io`
- **Service:** `k3s-agent.service`
- **Join token:** Retrieved from server node
- **Server URL:** `https://192.168.87.10:6443`

### k3s Components

k3s bundles:

- Kubernetes (control plane + kubelet)
- containerd (container runtime)
- CoreDNS
- Traefik (ingress controller)
- ServiceLB (basic LoadBalancer, replaced by MetalLB in our setup)
- Local-path storage provisioner

## Integration with Other Repos

### Relationship to homelab-gitops

**This repo:** Initial cluster provisioning (infrastructure layer)

**homelab-gitops:** Application deployment and management (GitOps layer)

**Workflow:**

1. Run `ansible-playbook` to provision cluster (this repo)
1. Install ArgoCD via Helm (bootstrap in homelab-gitops)
1. ArgoCD manages all subsequent deployments from Git

### Relationship to helm-charts

Not directly related. helm-charts contains custom Helm charts, but the cluster
uses OSS charts deployed via homelab-gitops.

## Network Configuration

### Cluster Network

- **Pod CIDR:** Default k3s (10.42.0.0/16)
- **Service CIDR:** Default k3s (10.43.0.0/16)
- **Node Network:** 192.168.87.0/24
- **MetalLB Pool:** 192.168.87.100-110 (configured post-deployment)

### Firewall/Ports

**k3s Server (192.168.87.10):**

- 6443: Kubernetes API
- 10250: Kubelet
- 2379-2380: etcd (embedded)

**k3s Agents:**

- 10250: Kubelet

**All nodes:**

- SSH: 22

## Pre-commit Hooks

This repository uses pre-commit hooks for quality control.

**Installed hooks:**

- ansible-lint
- yamllint
- check-yaml
- trailing whitespace, end-of-file
- markdownlint
- Conventional Commits

**Setup:**

```bash
pre-commit install              # One-time setup
pre-commit run --all-files      # Run manually (DO THIS BEFORE COMMITTING!)
pre-commit autoupdate           # Update hook versions
```

**IMPORTANT:** Always run `pre-commit run --all-files` BEFORE committing
to catch and fix errors (especially markdown and ansible-lint issues).

## Troubleshooting

### SSH Connection Issues

```bash
# Test SSH manually
ssh -i ~/.ssh/ansible_key ansible@hp-elitedesk

# Verify ansible can connect
ansible all -i inventory.ini -m ping

# Check SSH key permissions
ls -la ~/.ssh/ansible_key  # Should be 600
```

### k3s Installation Failures

```bash
# Check k3s service on server
ssh ansible@hp-elitedesk
sudo systemctl status k3s

# Check k3s logs
sudo journalctl -u k3s -f

# Reinstall k3s (if needed)
/usr/local/bin/k3s-uninstall.sh  # Server
/usr/local/bin/k3s-agent-uninstall.sh  # Agents
```

### Agent Join Issues

```bash
# Verify server is accessible from agent
ping 192.168.87.10
curl -k https://192.168.87.10:6443

# Check agent service
sudo systemctl status k3s-agent
sudo journalctl -u k3s-agent -f

# Verify node token
ssh ansible@hp-elitedesk sudo cat /var/lib/rancher/k3s/server/node-token
```

## Important Notes

### When Working on This Repo

1. **Test in VirtualBox first** - Use commented nodes for testing changes
1. **Run pre-commit hooks** BEFORE committing (fix all errors!)
1. **Backup before major changes** - k3s cluster data is critical
1. **Document role changes** - Update this file when roles change
1. **Use tags** - For targeted playbook runs (e.g., `--tags dependencies`)
1. **Idempotency** - Ensure roles can run multiple times safely

### Cluster Rebuild Process

If you need to rebuild the cluster:

1. Run uninstall scripts on all nodes
1. Clean up any remaining k3s data
1. Re-run ansible playbook
1. Re-bootstrap ArgoCD from homelab-gitops
1. ArgoCD will restore all applications from Git

### Secrets Management

- **SSH key:** Stored in 1Password (`k3s-node-ansible-ssh-key`)
- **k3s token:** Generated during server install, not stored in Git
- **Kubeconfig:** Retrieved from server, stored locally (not in Git)

## TODO / Future Improvements

- [ ] Setup and configure molecule tests
- [ ] Add role for NFS client configuration
- [ ] Add monitoring agent installation (Prometheus node-exporter)
- [ ] Automate kubeconfig retrieval post-install
- [ ] Add role for kernel parameter tuning
- [ ] Consider using Ansible Vault for sensitive variables

## References

- @README.md - Quick start guide
- k3s Documentation: <https://docs.k3s.io/>
- Ansible Documentation: <https://docs.ansible.com/>
- Related repo: @../homelab-gitops - GitOps deployment

---

**Last Updated:** 2025-11-14

This file should be updated whenever:

- Node inventory changes (new nodes added/removed)
- Roles are modified or added
- Network configuration changes
- k3s version is upgraded
- Important troubleshooting steps are discovered
