
# OpenStack Deployment

*Deploy a full OpenStack environment on resource-constrained hardware using containerization*

## Introduction

This guide walks you through deploying OpenStack on a laptop with limited resources (8GB RAM) using Kolla-Ansible in all-in-one mode. While performance will be constrained by available memory, this setup is perfect for learning, development, and testing OpenStack functionality.

## System Requirements

| Component | Minimum Specification |
|-----------|----------------------|
| Operating System | Ubuntu 22.04 LTS |
| Memory | 8 GB RAM |
| Storage | 200 GB free disk space |
| CPU | 4+ cores recommended |
| Network | Working internet connection |

## Deployment Architecture

Kolla-Ansible deploys OpenStack services as Docker containers on a single node, with these key components:

- **Keystone**: Identity service
- **Nova**: Compute service
- **Neutron**: Networking service
- **Glance**: Image service
- **Horizon**: Dashboard interface

## Installation Process

### Step 1: System Preparation

Update your system and install required dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-dev libffi-dev gcc libssl-dev python3-pip git -y
```

### Step 2: Docker Setup

Install and configure Docker:

```bash
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker  # Reload group permissions
```

### Step 3: Install Ansible and Kolla-Ansible

```bash
pip3 install -U pip
pip3 install 'ansible>=6,<8'
pip3 install kolla-ansible
```

### Step 4: Configure Kolla-Ansible

Create and prepare configuration directories:

```bash
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp /usr/local/share/kolla-ansible/ansible/inventory/all-in-one .
```

### Step 5: Generate Passwords

Generate random passwords for OpenStack services:

```bash
kolla-genpwd
```

> **Note**: You can optionally edit `/etc/kolla/passwords.yml` to set a custom admin password by modifying the `keystone_admin_password` value.

### Step 6: Configure Global Settings

Edit the global configuration file:

```bash
nano /etc/kolla/globals.yml
```

Add or modify these key settings:

```yaml
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "2023.1"
kolla_internal_vip_address: "127.0.0.1"
network_interface: "eth0"  # Run `ip a` to confirm your interface name
neutron_plugin_agent: "openvswitch"
enable_horizon: "yes"
enable_cinder: "no"  # Disable storage service to save RAM
enable_heat: "no"    # Disable orchestration to save RAM
enable_sahara: "no"  # Disable data processing to save RAM
```

### Step 7: Bootstrap Server

Prepare your system for OpenStack deployment:

```bash
kolla-ansible -i all-in-one bootstrap-servers
```

### Step 8: Run Prechecks and Pull Images

Verify your configuration and download container images:

```bash
kolla-ansible -i all-in-one prechecks
kolla-ansible -i all-in-one pull
```

> **Warning**: This step downloads approximately 5-10 GB of Docker images and may take considerable time depending on your internet connection.

### Step 9: Deploy OpenStack

Start the deployment process:

```bash
kolla-ansible -i all-in-one deploy
```

A successful deployment will end with a message indicating completion.

### Step 10: Configure OpenStack Client

Set up command-line access to your OpenStack environment:

```bash
kolla-ansible -i all-in-one post-deploy
source /etc/kolla/admin-openrc.sh
```

Verify functionality:

```bash
pip install python-openstackclient
openstack service list
```

## Accessing OpenStack

### Horizon Dashboard

Access the web interface:

1. Open a browser and navigate to: `http://<your-laptop-ip>/`
2. Log in using:
   - Username: `admin`
   - Password: Found in `/etc/kolla/passwords.yml` under `keystone_admin_password`

### Command Line

For CLI access after initial setup:

```bash
source /etc/kolla/admin-openrc.sh
openstack server list  # Example command
```

## Performance Optimization

### Memory-Saving Tips

| Strategy | Implementation |
|----------|----------------|
| Disable unused services | Set `enable_[service]` to "no" in globals.yml |
| Limit container resources | Add CPU/memory constraints to Docker |
| Run minimal services | Keep only Keystone, Nova, Neutron, Glance, Horizon |
| Reduce logging | Configure less verbose logging levels |

### Recommended Service Configuration

For 8GB RAM systems, consider this configuration in `globals.yml`:

```yaml
# Core services
enable_keystone: "yes"
enable_nova: "yes"
enable_neutron: "yes"
enable_glance: "yes"
enable_horizon: "yes"

# Optional services - disable to save memory
enable_cinder: "no"
enable_swift: "no"
enable_heat: "no"
enable_ceilometer: "no"
enable_gnocchi: "no"
enable_sahara: "no"
enable_designate: "no"
enable_freezer: "no"
enable_watcher: "no"
```

## Cleanup

To remove the deployment and all data:

```bash
kolla-ansible -i all-in-one destroy --yes-i-really-really-mean-it
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Deployment fails due to memory | Disable more services in globals.yml |
| Network connectivity issues | Check `network_interface` setting matches your system |
| Container startup failures | Check Docker logs: `docker logs <container_name>` |
| Dashboard unreachable | Verify IP address and that Horizon container is running |

