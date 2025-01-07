# ansible-node-setup

![GitHub branch status](https://img.shields.io/github/checks-status/mosher-labs/ansible-node-setup/main)
![GitHub Issues](https://img.shields.io/github/issues/mosher-labs/ansible-node-setup)
![GitHub last commit](https://img.shields.io/github/last-commit/mosher-labs/ansible-node-setup)
![GitHub repo size](https://img.shields.io/github/repo-size/mosher-labs/ansible-node-setup)
![Libraries.io dependency status for GitHub repo](https://img.shields.io/librariesio/github/mosher-labs/ansible-node-setup)
![GitHub License](https://img.shields.io/github/license/mosher-labs/ansible-node-setup)
![GitHub Sponsors](https://img.shields.io/github/sponsors/mosher-labs)

## ⚙️  K8s Node Configuration with Ansible 🌐

Welcome to the K3s Kubernetes Node Setup repository! 🚀 This repo
provides Ansible playbooks and roles designed to configure and
manage nodes for lightweight Kubernetes clusters using K3s. 🎯

🌟 Key Features:

- 📜 Automated node provisioning and configuration for K3s clusters.
- 🔧 Support for common Kubernetes-ready optimizations and settings.
- 🖧 Seamless integration with existing Ansible workflows.

✨ Perfect for:

- Setting up development, testing, or production K3s clusters 🚀
- Managing scalable Kubernetes nodes with ease 🛠️
- Ensuring consistency and reproducibility across your infrastructure 🌍
- Clone the repo, run the playbooks, and get your cluster ready in no time! 🤝

## Usage

### 📦 Dependencies

```bash
mise install pipx
pipx install --incude-deps ansible
pipx ensurepath
```

### Configure individual node

After basic Ubuntu Server install:

```bash
ssh ubuntu-virtualbox
sudo adduser ansible
sudo usermod -aG sudo ansible
sudo su
echo 'ansible ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
exit
exit

# On host machine

# Copy SSH key to node
ssh-copy-id ansible@ubuntu-virtualbox

# Export the key to be used by Ansible
op item get "k3s-node-ansible-ssh-key" --field "private key" > $HOME/.ssh/ansible_key
```

### Run ansible playbooks

```bash
ansible-galaxy collection install --requirements-file requirements.yml
ansible-playbook -i inventory.ini playbook.yml \
--private-key ~/.ssh/ansible_key --user ansible
```

### Setup kubeconfig

```bash
scp -i ~/.ssh/ansible_key ansible@192.168.202.14:/etc/rancher/k3s/k3s.yaml ~/k3s.yaml
sed -i '' 's/127.0.0.1/192.168.202.14/g' ~/k3s.yaml
export KUBECONFIG=~/k3s.yaml
kubectl get nodes
kubectl get deployments --namespace default
kubectl get pods --namespace default
kubectl get services --namespace default
```

## 🔰 Contributing

Upon first clone, install the pre-commit hooks.

```bash
pre-commit install
```

To run pre-commit hooks locally, without a git commit.

```bash
pre-commit run -a --all-files
```

To update pre-commit hooks, this ideally should be ran before a pull request is merged.

```bash
pre-commit autoupdate
```

### 📋 TODO

- [ ] Setup and configure molecule tests
