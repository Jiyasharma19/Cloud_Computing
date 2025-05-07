
# FRP Server Deployment- AWS EC2

> *Fast Reverse Proxy (FRP) allows you to expose local services behind NAT or firewalls to the internet*

## Overview

This guide provides instructions for deploying an FRP server on an Ubuntu-based AWS EC2 instance to securely expose internal services to the public internet.

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Cloud Platform | AWS EC2 |
| Operating System | Ubuntu 20.04/22.04 LTS |
| Access | SSH key pair |
| Knowledge | Basic Linux commands |
| Client | FRP client ready on your device |

## Deployment Steps

### 1️⃣ EC2 Instance Setup

**Launch Configuration:**
- **Image**: Ubuntu 22.04 LTS
- **Size**: t2.micro (minimum)
- **Storage**: 8+ GB recommended
- **Security Group Rules**:
  - SSH (22): Your IP
  - FRP Server (7000): Any source
  - Dashboard/Services (7500-7600): Any source

**Connect to your instance:**
```bash
ssh -i your-key.pem ubuntu@<ec2-public-ip>
```

### 2️⃣ FRP Installation

**Download and extract:**
```bash
wget https://github.com/fatedier/frp/releases/download/v0.58.0/frp_0.58.0_linux_amd64.tar.gz
tar -xvzf frp_0.58.0_linux_amd64.tar.gz
cd frp_0.58.0_linux_amd64
```

**Server Configuration:**

Create a `frps.ini` file with the following content:

```ini
[common]
bind_port = 7000
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = changeme123

# Optional: for HTTP/HTTPS services
vhost_http_port = 8080
vhost_https_port = 8443
```

### 3️⃣ Server Operation

**Testing mode:**
```bash
./frps -c frps.ini
```

**Background operation:**
```bash
nohup ./frps -c frps.ini &
```

> **Pro Tip**: Consider setting up a systemd service for automatic startup

### 4️⃣ Client Configuration

On your local machine:

1. Download matching FRP version
2. Create `frpc.ini`:

```ini
[common]
server_addr = <your-ec2-public-ip>
server_port = 7000

[expose-ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

3. Launch the client:
```bash
./frpc -c frpc.ini
```

4. Test connection:
```bash
ssh -p 6000 user@<ec2-public-ip>
```

## Security Best Practices

- ✅ Use strong credentials for the dashboard
- ✅ Implement IP restrictions when possible
- ✅ Regularly update FRP to the latest version
- ✅ Only open necessary ports in your security group

## Additional Resources

- [FRP GitHub Repository](https://github.com/fatedier/frp)
- [FRP Documentation](https://github.com/fatedier/frp/blob/dev/README.md)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)

---


