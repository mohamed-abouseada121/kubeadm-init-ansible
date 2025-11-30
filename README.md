# kubeadm-init-ansible

An Ansible automation framework for deploying a production-ready Kubernetes cluster using `kubeadm` with containerd as the container runtime.

## Overview

This project automates the complete setup of a Kubernetes cluster with:
- **Master Node**: Control plane initialization with Flannel CNI
- **Worker Nodes**: Automated joining to the cluster
- **Container Runtime**: Containerd with systemd cgroup driver
- **Kubernetes v1.29**: Latest stable version with proper GPG key management

## Prerequisites

- Debian/Ubuntu-based systems (tested on Ubuntu 20.04+)
- SSH access to all target nodes
- Root or sudo privileges on target nodes
- Ansible 2.9+ installed on the control machine

## Project Structure

```
kubeadm-init-ansible/
├── ansible.cfg                 # Ansible configuration
├── README.md                   # This file
├── inventory/
│   ├── hosts.ini              # Host inventory
│   └── group_vars/
│       └── all.yml            # Group variables
├── playbooks/
│   ├── install_kubeadm_master.yml    # Master node playbook
│   └── install_kubeadm_workers.yml   # Worker nodes playbook
└── roles/
    ├── master-role/           # Master node role
    │   ├── handlers/
    │   │   └── main.yml
    │   └── tasks/
    │       └── main.yml
    └── worker-role/           # Worker node role
        ├── handlers/
        │   └── main.yml
        └── tasks/
            └── main.yml
```

## Quick Start

### 1. Configure Inventory

Edit [`inventory/hosts.ini`](inventory/hosts.ini) with your node details:

```ini
[kubeadm-master]
kube-master1 ansible_host=<master-ip> ansible_user=ubuntu

[kubeadm-workers]
kube-worker1 ansible_host=<worker1-ip> ansible_user=ubuntu
kube-worker2 ansible_host=<worker2-ip> ansible_user=ubuntu
```

### 2. (Optional) Update Variables

Customize variables in [`inventory/group_vars/all.yml`](inventory/group_vars/all.yml) if needed.

### 3. Run the Playbooks

**Install master node:**
```bash
ansible-playbook playbooks/install_kubeadm_master.yml
```

**Install worker nodes:**
```bash
ansible-playbook playbooks/install_kubeadm_workers.yml
```

**Or run both:**
```bash
ansible-playbook playbooks/install_kubeadm_master.yml playbooks/install_kubeadm_workers.yml
```

## What Gets Installed

### System Configuration
- Disables swap (required for Kubernetes)
- Loads required kernel modules (`br_netfilter`, `overlay`)
- Configures sysctl parameters for networking

### Packages
- **containerd**: Container runtime
- **kubeadm**: Cluster initialization tool
- **kubelet**: Node agent
- **kubectl**: Command-line tool
- **docker.io**: Docker utilities

### Kubernetes Setup
- **Master**: Initializes cluster with Pod CIDR `10.244.0.0/16`
- **CNI**: Installs Flannel for pod networking
- **Workers**: Joins workers to the cluster automatically

## Key Features

✅ **Automated GPG Key Management**: Adds and verifies Kubernetes repository keys  
✅ **Containerd Runtime**: Pre-configured with systemd cgroup driver  
✅ **Persistent Configuration**: Kernel module and sysctl changes persist across reboots  
✅ **Fact Caching**: Ansible caches facts for faster subsequent runs  
✅ **Worker Auto-Join**: Workers automatically join using the join token from master  

## Cluster Verification

After playbooks complete, verify your cluster on the master node:

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

## Networking

- **Pod CIDR**: `10.244.0.0/16` (Flannel default)
- **Service CIDR**: Default Kubernetes range
- **CNI Plugin**: Flannel

## Troubleshooting

**Workers not joining?**
- Ensure master node initialization completed successfully
- Check network connectivity between nodes
- Verify firewall rules allow Kubernetes ports (6443, 10250, etc.)

**Containerd issues?**
- Check handler execution: `systemctl status containerd`
- Verify config: `cat /etc/containerd/config.toml`

**Join command expired?**
- Generate a new token on master: `kubeadm token create --print-join-command`

## Configuration Files

- [`ansible.cfg`](ansible.cfg): Ansible settings (inventory path, roles path, caching)
- [`inventory/hosts.ini`](inventory/hosts.ini): Host inventory
- [`roles/master-role/tasks/main.yml`](roles/master-role/tasks/main.yml): Master setup tasks
- [`roles/worker-role/tasks/main.yml`](roles/worker-role/tasks/main.yml): Worker setup tasks

## Notes

- Kubernetes packages are held at their installed versions to prevent automatic updates
- The join command is generated dynamically on each run
- Fact caching speeds up repeated playbook executions
- Ensure SSH key-based authentication is configured

## License

MIT

## Support

For issues or improvements, please review the task files and ensure all prerequisites are met.
