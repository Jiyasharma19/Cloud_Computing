
# OpenStack on EC2 Setup

*A step-by-step tutorial for installing OpenStack with DevStack on Amazon EC2*

## Introduction

This guide walks you through the process of setting up OpenStack using DevStack on an Ubuntu-based EC2 instance. Once completed, you'll have access to the OpenStack Dashboard (Horizon) where you can create and manage cloud resources.

## System Requirements

| Component | Specification |
|-----------|---------------|
| Base Image | Ubuntu 22.04 LTS |
| Instance Size | t2.large (2 vCPUs, 8GB RAM) or larger |
| Storage | 30+ GB |
| Network Access | SSH (22), HTTP (80), HTTPS (443) |

## Security Configuration

Create a security group with the following inbound rules:

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | Your IP | SSH access |
| 80 | TCP | 0.0.0.0/0 | HTTP (Dashboard) |
| 443 | TCP | 0.0.0.0/0 | HTTPS (Dashboard) |

## Installation Procedure

### Step 1: Access Your Instance

Connect to your EC2 instance using SSH:

```bash
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
```

### Step 2: System Preparation

Update package repositories and install required dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-pip python3-dev net-tools
```

### Step 3: Get DevStack Source

Clone the DevStack repository:

```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

### Step 4: Configure Installation

Create a `local.conf` configuration file:

```bash
nano local.conf
```

Add the following content:

```ini
[[local|localrc]]
ADMIN_PASSWORD=SuperSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
HOST_IP=$(hostname -I | awk '{print $1}')
disable_service tempest
disable_service cinder
disable_service swift
disable_service heat
USE_PYTHON3=True
```

### Step 5: Install OpenStack

Start the installation process:

```bash
./stack.sh
```

> **Note**: Installation takes approximately 30-45 minutes. Do not interrupt this process.

### Step 6: Access the Dashboard

After successful installation:

1. Open your web browser
2. Navigate to `http://<EC2-PUBLIC-IP>/dashboard`
3. Log in with:
   - Username: `admin`
   - Password: `SuperSecret` (or as defined in your config)

### Step 7: Verify Deployment

Check that all services are running:

```bash
source openrc admin admin
openstack service list
```

A successful deployment will show core services like Nova, Glance, Neutron, and Keystone.

## Service Management

If you need to restart a service:

```bash
sudo systemctl restart devstack@<service-name>
```

Example:
```bash
sudo systemctl restart devstack@keystone
```

## Important Limitations

- This installation is suitable for **development and testing only**
- Some OpenStack features may be limited by EC2's virtualization architecture
- Performance will vary based on instance type
- Not recommended for production workloads

## Troubleshooting

| Issue | Resolution |
|-------|------------|
| Service not starting | Check logs in `/var/log/devstack` |
| Dashboard inaccessible | Verify security group settings |
| Instance creation fails | Ensure nested virtualization is supported |

