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
