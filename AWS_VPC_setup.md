# AWS VPC Setup
> *Creating a multi-AZ architecture with public and private subnets*

## Architecture Overview

This guide demonstrates how to build an AWS Virtual Private Cloud (VPC) infrastructure with the following components:

| Component | Details |
|-----------|---------|
| VPC | Single VPC with CIDR block `10.0.0.0/24` |
| Availability Zones | Two zones (A and B) for high availability |
| Subnets | Four total: two public and two private subnets |
| Connectivity | Internet Gateway (IGW) for public internet access |
| | Virtual Private Gateway (VPG) for private subnet connectivity |

### Network Configuration

| Subnet | CIDR Range | Availability Zone | Access |
|--------|------------|-------------------|--------|
| Public Subnet 1 | `10.0.0.0/26` | Zone A | Internet via IGW |
| Public Subnet 2 | `10.0.0.64/26` | Zone B | Internet via IGW |
| Private Subnet 1 | `10.0.0.128/26` | Zone A | Internal via VPG |
| Private Subnet 2 | `10.0.0.192/26` | Zone B | Internal via VPG |

## Key Concepts

### Virtual Private Cloud (VPC)
A logically isolated section of AWS that lets you provision resources in a defined virtual network with complete control over IP addressing, routing, security, and network configuration.

### Subnets
Segments of IP address ranges within your VPC, allowing you to group resources by purpose and location. Public subnets route to the internet, while private subnets remain isolated.

### Gateways
* **Internet Gateway (IGW)**: Allows resources in public subnets to connect to the internet
* **Virtual Private Gateway (VPG)**: Enables secure communication between private subnets and other networks

### Route Tables
Collections of rules (routes) that determine where network traffic from your subnet is directed.

## Implementation Guide

### Step 1: Create the VPC

1. Navigate to the AWS Management Console → VPC Dashboard
2. Select **Create VPC**
3. Configure the following settings:
   * Name: `My-VPC`
   * IPv4 CIDR block: `10.0.0.0/24`
   * Leave remaining options as default
4. Click **Create VPC**

### Step 2: Create Subnets

#### For Zone A:

1. In the VPC Dashboard, navigate to **Subnets** → **Create Subnet**
2. Configure public subnet:
   * Name: `Public-Subnet-1`
   * VPC: `My-VPC`
   * Availability Zone: Select first AZ (e.g., `us-east-1a`)
   * IPv4 CIDR block: `10.0.0.0/26`
   * Click **Create Subnet**

3. Configure private subnet:
   * Name: `Private-Subnet-1`
   * VPC: `My-VPC`
   * Availability Zone: Same as public subnet
   * IPv4 CIDR block: `10.0.0.128/26`
   * Click **Create Subnet**

#### For Zone B:

1. Create public subnet:
   * Name: `Public-Subnet-2`
   * VPC: `My-VPC`
   * Availability Zone: Select second AZ (e.g., `us-east-1b`)
   * IPv4 CIDR block: `10.0.0.64/26`

2. Create private subnet:
   * Name: `Private-Subnet-2`
   * VPC: `My-VPC`
   * Availability Zone: Same as second public subnet
   * IPv4 CIDR block: `10.0.0.192/26`

### Step 3: Configure Internet Gateway

1. Navigate to **Internet Gateways** → **Create Internet Gateway**
2. Name: `My-IGW`
3. Click **Create**
4. Select the new IGW → **Actions** → **Attach to VPC**
5. Select `My-VPC` and attach

### Step 4: Set Up Virtual Private Gateway

1. Navigate to **Virtual Private Gateways** → **Create Virtual Private Gateway**
2. Name: `My-VPG`
3. Leave ASN as default
4. Click **Create**
5. Select the new VPG → **Actions** → **Attach to VPC**
6. Select `My-VPC` and attach

### Step 5: Configure Route Tables

#### Public Route Table:

1. Navigate to **Route Tables** → **Create Route Table**
2. Name: `Public-Route-Table`
3. VPC: `My-VPC`
4. Click **Create**
5. Configure internet access:
   * Select the route table → **Routes** tab → **Edit Routes**
   * Add route with:
     * Destination: `0.0.0.0/0`
     * Target: `My-IGW`
   * Click **Save changes**
6. Associate with public subnets:
   * Select **Subnet Associations** tab → **Edit subnet associations**
   * Select both `Public-Subnet-1` and `Public-Subnet-2`
   * Click **Save associations**

#### Private Route Table:

1. Create another route table:
   * Name: `Private-Route-Table`
   * VPC: `My-VPC`
2. Enable VPG connectivity:
   * Select the route table → **Route Propagation** tab
   * Click **Edit route propagation**
   * Select `My-VPG` and enable propagation
   * Click **Save**
3. Associate with private subnets:
   * Select **Subnet Associations** tab → **Edit subnet associations**
   * Select both `Private-Subnet-1` and `Private-Subnet-2`
   * Click **Save associations**

### Step 6: Deploy EC2 Instances

#### In Public Subnets:

1. Navigate to EC2 Dashboard → **Launch Instance**
2. Configure instance:
   * Choose AMI (e.g., Amazon Linux 2)
   * Select instance type (e.g., `t2.micro`)
   * Configure network:
     * VPC: `My-VPC`
     * Subnet: `Public-Subnet-1`
     * Auto-assign Public IP: Enable
   * Configure security group to allow required access
   * Create or select an existing key pair
   * Launch instance
3. Repeat for `Public-Subnet-2`

#### In Private Subnets:

1. Launch instance with similar configuration but:
   * Subnet: `Private-Subnet-1`
   * Auto-assign Public IP: Disable
   * Security group allowing only internal access
2. Repeat for `Private-Subnet-2`

## Verification

### Testing Connectivity

| Test | Method | Expected Result |
|------|--------|----------------|
| Public subnet internet access | SSH to instance and run `ping google.com` | Successful pings |
| Private subnet isolation | Try to access private instance from internet | Connection timeout |
| Private subnet internal communication | SSH to public instance, then to private instance, then ping other private instance | Successful pings |

## Troubleshooting

| Issue | Potential Solution |
|-------|-------------------|
| Cannot connect to public instance | Check security group rules and route table configuration |
| Public instance cannot access internet | Verify IGW attachment and route table settings |
| Private instances cannot communicate | Check VPG configuration and route propagation |
| Cannot SSH to private instance from public | Ensure security groups allow traffic between subnets |


